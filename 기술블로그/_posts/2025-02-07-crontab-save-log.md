---
layout: post  
title: "crontab으로 로그 저장하기"
author: "김규영"
banner:
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["crontab", "docker"]
---
### 시작
> 앱의 로그를 파일로 관리하라는 피드백을 받았습니다.
>
> 현재 애플리케이션을 도커에서 실행중이기 때문에 도커 외부에 저장해야합니다.
>
> 왜냐면 컨테이너의 데이터는 휘발성이니까요
>
> 앱 내부에서 파일로 저장하는게 더 좋은 방법인 것 같지만 이걸 할 당시에는 내부에서는 저장하지 않고 있었어요
>
> 서버에 들어가지 않고 로그를 확인할 수 있는 수단이 필요했기 때문에 일단 crontab과 shell 스크립트를 사용해서 로그를 저장하기로 했습니다.

- 구현 목표
  - 1분마다 한 번씩 로그를 출력하고 저장
  - 이전에 있는 로그와 중복되지 않도록 한다.
  - 로그는 하루에 한번 다른 파일로 교체한다.
  - 로그가 없으면 저장하지 않는다.

<br>

- 일단 crontab에 대해 대충 설명을 해보면
  - 일정 시간마다 지정한 동작을 수행하도록 합니다.
    - \* * * * * echo "123123" 처럼 작성합니다
      - 이건 1분에 한번씩 동작을 수행하는걸 의미합니다.
    - 왼쪽부터  *은 분, 시, 일, 월, 요일을 의미합니다.
      - 각 자리의 * 은 매분, 매시, 매일, 매월, 모든 요일을 의미하며
        - \* 1 * * * 은 매일 01시에 수행

---
## crontab 설정
- 환경 : ubuntu 24.10
> crontab은 기본적으로 깔려있기 때문에 설치 할 필요가 없습니다.



````shell
 crontab - e 
````
이 명령어를 사용해서 crontab 설장 파일로 들어갑니다.

```
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command

```
- 파일로 들어가면 이런 장면이 나옵니다
- 영어는 어렵지만 긴장하지 말고 목적만 달성하고 나올겁니다
```
0 1 * * * /home/logs/main/rotate-logs.sh # 매일 1시에 수행
* * * * * /home/logs/main/log-reloader.sh # 1분마다 수행
```
- 일단 구현 목표인 매일 새로운 파일을 생성하기 위해 0 1 * * * 로 추가를 했어요.
- 그리고 1분마다 한 번씩 로그를 갱신하기 위해 * * * * *도 추가를 합니다.
- 뒤에 붙어있는 경로는 쉘 스크립트의 경로입니다 이제 만들러 갈겁니다.
---
## shell script 작성
- crontab 설정을 해놨으니 이제 지정한 시간마다 계속 동작을 수행할거에요.
- 당연히 shell script는 안만들었으니까 crontab은 계속 헛수고만 하고있겠죠


````shell
#!/bin/bash
#### log-reloader.sh
# 로그 저장 디렉터리
LOG_DIR="/home/logs/main"
mkdir -p "$LOG_DIR"

LOG_FILE="$LOG_DIR/main-latest.log"

# 마지막 로그 타임스탬프 저장 파일
TIMESTAMP_FILE="$LOG_DIR/last_timestamp"


# 마지막 실행 이후의 로그만 가져오기
if [[ -f "$TIMESTAMP_FILE" ]]; then
    LAST_TIMESTAMP=$(cat "$TIMESTAMP_FILE")
    docker logs --since "$LAST_TIMESTAMP" main >> "$LOG_FILE" 2>&1
else
    docker logs main > "$TMP_LOG_FILE" 2>&1 # 실행한 적 없으면 타임 스탬프가 비어있음
fi


# 현재 타임스탬프 저장 (다음 실행을 위해)
date --utc +%FT%T > "$TIMESTAMP_FILE"
````
- 1분마다 로그를 갱신하는 스크립트입니다.
- 로그 저장 디렉토리를 만들고 타임 스탬프 기준으로 도커에서 로그를 빼옵니다.
- 마지막에는 항상 실행한 시간을 갱신해줍니다. 그래야 다음 실행에서 이 시간을 기준으로 도커에서 로그를 출력해 가져옵니다.

```shell
#!/bin/bash
# rotate-logs.sh
# 로그 저장 디렉터리
LOG_DIR="/home/logs/main"
mkdir -p "$LOG_DIR"

# 새로운 로그 파일 이름 (형식: main-YYYYMMDDHHMMSS.log)
NEW_LOG_FILE="$LOG_DIR/main-$(date +'%Y%m%d%H%M%S').log"
# 마지막 로그 타임스탬프 저장 파일
TIMESTAMP_FILE="$LOG_DIR/last_timestamp"

# 새로운 로그를 임시 파일에 저장
TMP_LOG_FILE=$(mktemp)

# 마지막 실행 이후의 로그만 가져오기
# 타임스탬프가 있으면 이전에 한 번 실행이 되었기 때문에 로그가 있음
if [[ -f "$TIMESTAMP_FILE" ]]; then
    LAST_TIMESTAMP=$(cat "$TIMESTAMP_FILE")
    cat "$LOG_DIR/main-latest.log" > "$TMP_LOG_FILE" 2>&1
else
    date --utc +%FT%T > "$TIMESTAMP_FILE" # 타임스탬프 업데이트
    docker logs main > "$TMP_LOG_FILE" 2>&1 # 만약 없으면 전체 로그 저장
fi

# 로그가 비어 있는지 확인
if [[ -s "$TMP_LOG_FILE" ]]; then
    # 심볼릭 링크를 최신 로그 파일로 업데이트
    ln -sf "$NEW_LOG_FILE" "$LOG_DIR/main-latest.log"
else
    # 로그가 없으면 임시 파일 삭제
    rm "$TMP_LOG_FILE"
fi                                                                                                                                                                                                                                                                                                                                    
```
- 하루에 한번 새로운 로그파일로 교체하는 스크립트입니다.
- 여기서는 타임 스탬프가 없을 때 빼고는 타임스탬프를 업데이트 하지 않습니다.
---
## 완성
> 이제 crontab이 계속 지장한 시간마다 shell script를 실행하며 새로운 로그를 생성 할겁니다.

### 단점
```shell
# 마지막 실행 이후의 로그만 가져오기
if [[ -f "$TIMESTAMP_FILE" ]]; then
    LAST_TIMESTAMP=$(cat "$TIMESTAMP_FILE")
    docker logs --since "$LAST_TIMESTAMP" main >> "$LOG_FILE" 2>&1
else
    docker logs main > "$TMP_LOG_FILE" 2>&1 # 실행한 적 없으면 타임 스탬프가 비어있음
fi
######### 로그 100개 ############

# 현재 타임스탬프 저장 (다음 실행을 위해)
date --utc +%FT%T > "$TIMESTAMP_FILE"
```
- 임시 방편인 만큼 약간 문제가 좀 있습니다.
  - 로그를 업데이트하고 타임스탬프를 업데이트 하기전 찰나의 순간에 발생된 로그가 무시 될 가능성이 있습니다.
  - 약간의 중복을 허용하는걸로 해결 할 수 있지만 그냥 넘어가기로 했습니다.
- 여기에 쏟은 시간만큼 정이 들었지만 아쉽게도 조만간 버려야 할 것 같습니다,

- 그냥 docker의 volume을 사용 하세요

---

---
layout: post
title: "도커 컨테이너"
author: "김민규"
categories: "기술세미나"
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: ["docker", "container"]
---

# 도커 컨테이너

안녕하세요. kernel360 크루 김민규입니다.

이번 글에선 도커 컨테이너에 대해 이야기해보겠습니다.

### 컨테이너 = 리눅스 프로세스

도커 컨테이너는 가상머신이 아니라 리눅스 프로세스입니다.

도커는 리눅스의 커널이 제공하는 기술을 활용합니다. 

> **“컨테이너는 리눅스 프로세스이며, 도커는 리눅스 커널이 제공하는 기능을 사용하는 API 데몬이다.”**
> 
> 
> ![image](https://github.com/user-attachments/assets/1cb93685-e219-476f-af18-3f2413ddfa6f)
>
> <[Red Hat 컨퍼런스 발표](https://www.youtube.com/watch?v=KawKGsLR1V8&list=PLRG5QO3z4WLSElnCHCX5TyiJpSU4AV_Hr) 영상>
> 

그래서 윈도우, 맥에서는 다음의 과정을 거쳐 도커가 실행됩니다.

![image](https://github.com/user-attachments/assets/1caf93d4-20ef-4d80-aec4-ab6231c1ec1d)

1. 윈도우나 맥OS의 하이퍼바이저 가상화 기술을 사용하여 윈도우/맥OS에서 리눅스 가상머신을 실행.
2. 실행된 `리눅스` 가상 머신 커널을 사용해서 컨테이너 환경을 구성합니다.

컨테이너는 리눅스 커널이 제공하는 아래 2가지 기능을 이용해 격리됩니다.

### (1) Cgroups

`프로세스가 자원을 얼마나 사용하게 할지 결정한다` 

![image](https://github.com/user-attachments/assets/24d36e65-f975-4b50-af44-9080afca7e89)

cgroups는 프로세스들의 자원 사용(cpu, 메모리, 네트워크)를 제한하고 격리시키는 Linux 커널 기능입니다. 

cgroups를 이용하면, 하드웨어 자원의 사용 범위를 어플리케이션마다 다르게 할당할 수 있습니다. ([리눅스 메뉴얼](https://man7.org/linux/man-pages/man7/cgroups.7.html))

### (2) Namespace

![image](https://github.com/user-attachments/assets/4d9efcd5-9df9-49c2-af68-1fc6b6dda5c8)

`프로세스가 어떤 자원을 볼 수 있는지 결정한다`

프로세스의 다양한 측면을 격리하는 Linux 커널 기능입니다.

한 프로세스 집합은 한 리소스 집합을 보고 다른 프로세스 집합은 다른 리소스 집합을 보도록 리소스를 분할하는 기능입니다. ([리눅스 메뉴얼](https://man7.org/linux/man-pages/man7/namespaces.7.html))

- 네임스페이스는 mount, uts, ipc, pid, neet, user까지 6가지 종류가 있습니다.
- `/proc/<pid>/ns` 경로로 직접 확인할 수 있습니다.
    
    ![image](https://github.com/user-attachments/assets/b6dcdf6e-8534-478c-81db-d18a915c87fc)

- `nsenter` 명령어로 특정 네임스페이스에 접속할 수 있습니다.
    
    ![image](https://github.com/user-attachments/assets/6b7c163b-8222-4b9a-8fa2-1508143ee7ce)

위 6가지 네임스페이스 중, 2가지만 이야기해보겠습니다.

1. **마운트 네임스페이스**
    
    ![image](https://github.com/user-attachments/assets/a6db3781-27f1-48b6-bd7e-549738de7d55)

    파일 시스템 마운트 포인트 격리하는 기능입니다.
    
    - 프로세스는 각자의 root 파일 시스템 (=`chroot` 와 유사)을 가질 수 있습니다.
    - 프로세스는 각자만의 Private 마운트를 가질 수도, 공유된 마운트를 가질 수 도 있습니다.
    
    ![image](https://github.com/user-attachments/assets/8e9be0e5-af17-45f4-9bd1-12fd991e58d2)

    도커 컨테이너의 마운트는 커널의 mnt 네임스페이스 기능입니다.
    
    따라서 도커 컨테이너에 `exec -it bin/bash`  명령어로 접속한 뒤, ls 명령어를 실행한 결과는 namesapace로 접속한 마운트 경로와 실제로 동일합니다.
    
    ![image](https://github.com/user-attachments/assets/e93db006-b3dd-49be-add5-26e2c0e5d203)

2. **PID 네임스페이스**
    
    PID (Process ID) Numberspace를 격리하는 기능입니다.
    
    ![image](https://github.com/user-attachments/assets/ec59b83f-8c8e-436e-b164-0b158875eabb)

    - 특정 PID 네임스페이스에 속한 프로세스는 같은 네임스페이스에 속한 프로세스만 볼 수 있습니다.
    - 각 PID 네임스페이스마다 1번 PID를 가집니다.
    - PID 1 프로세스가 종료되면, 전체 네임스페이스가 종료됩니다.
    - PID 네임스페이스는 중첩된 구조를 가질 수 있기에, 여러개의 PID 를 갖게 됩니다. (네임스페이스당 1개씩, 여러개)
        
     ![image](https://github.com/user-attachments/assets/0629994f-f1ce-4011-b24b-084dfc99c64c)

    
    1번 PID를 가진 프로세스는 특별한 프로세스입니다. pid 1은 `init 프로세스`로 커널이 생성합니다.
    
    ![image](https://github.com/user-attachments/assets/5cbe3104-6a1a-4358-9a3e-a0c82c7e714d)

    다음은 1번 프로세스의 특징입니다.
    
    1. 시그널 처리 기능 (커널이 보내는 시그널을 자식 프로세스에게 전파)
    2. 좀비, 고아 프로세스 정리
    3. 1번 프로세스가 죽으면 시스템 패닉 -> reboot 으로만 해결 가능
    
    도커 프로세스는 자신의 PID 네임스페이스를 갖게 됩니다.
    
    따라서 컨테이너 내부로 접속해서 ps 명령어를 실행하면, 컨테이너 실행 태스크가 곧 PID 1번을 갖게 됩니다.
    
    ![image](https://github.com/user-attachments/assets/c93ed973-689f-4744-8ffa-fe5b28f31022)

    우리가 컨테이너 Stop 명령어를 실행하면, 도커는 해당 PID 네임스페이스의 1번에게 종료 시그널을 보냅니다. 이로 인해 PID 1번이 종료되고, 해당 네임스페이스가 닫힙니다.
    
    ![image](https://github.com/user-attachments/assets/882bc866-60c5-4038-b27b-9832b33a7f3a)

    이때 주의할 점은, Init 프로세스와 다르게 컨테이너의 PID 1번은 자식 프로세스에게 종료 시그널을 전파하지 않는다는 점입니다. 따라서 컨테이너를 생성한 개발자가 직접 시그널을 전파하는 작업을 해줘야 합니다.
    
    ![image](https://github.com/user-attachments/assets/db6de3d3-95c7-4730-8e9f-d9ab73f7ac66)

    결국 도커는 리눅스의 기능을 활용하는 api 데몬이며, 내부적으로 리눅스 커널이 제공하는 `Cgroups`와 `Namespace` 기능을 이용해 컨테이너를 생성합니다.
    
    이번 글에서 도커가 내부적으로 어떻게 컨테이너를 생성하는지 간략히 소개해보았습니다.
    
    앞으로 프로젝트를 하며 도커를 사용함에 있어 작은 도움이 되셨길 바라며 글을 마치겠습니다.

![image](https://github.com/user-attachments/assets/2805ae5e-d1b2-4324-b17b-fb9c08ffc380)---
layout: post
title: 도커 컨테이너
author: 김민규
categories: 기술세미나
banner:
  image: "assets/images/post/2023-11-05.webp"
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
tags: [docker, container, 기술세미나]
---
# 도커 컨테이너

안녕하세요. kernel360 크루 김민규입니다.

이번 글에선 도커 컨테이너에 대해 이야기해보겠습니다.

### 컨테이너 = 리눅스 프로세스

도커 컨테이너는 가상머신이 아니라 리눅스 프로세스입니다.

도커는 리눅스의 커널이 제공하는 기술을 활용합니다. 

> **“컨테이너는 리눅스 프로세스이며, 도커는 리눅스 커널이 제공하는 기능을 사용하는 API 데몬이다.”**
> 
> ![image](https://github.com/user-attachments/assets/8747a956-fca9-4437-bea4-0b03cd55b2a9)
> 
> <[Red Hat 컨퍼런스 발표](https://www.youtube.com/watch?v=KawKGsLR1V8&list=PLRG5QO3z4WLSElnCHCX5TyiJpSU4AV_Hr) 영상>
> 

그래서 윈도우, 맥에서는 다음의 과정을 거쳐 도커가 실행됩니다.

![image](https://github.com/user-attachments/assets/5459535a-f4da-48d2-8b6e-8588136901ab)

1. 윈도우나 맥OS의 하이퍼바이저 가상화 기술을 사용하여 윈도우/맥OS에서 리눅스 가상머신을 실행.
2. 실행된 `리눅스` 가상 머신 커널을 사용해서 컨테이너 환경을 구성합니다.

컨테이너는 리눅스 커널이 제공하는 아래 2가지 기능을 이용해 격리됩니다.

### (1) Cgroups

`프로세스가 자원을 얼마나 사용하게 할지 결정한다` 

![image](https://github.com/user-attachments/assets/a4fe79be-6d94-4365-b8c3-edd8ed19c492)

cgroups는 프로세스들의 자원 사용(cpu, 메모리, 네트워크)를 제한하고 격리시키는 Linux 커널 기능입니다. 

cgroups를 이용하면, 하드웨어 자원의 사용 범위를 어플리케이션마다 다르게 할당할 수 있습니다. ([리눅스 메뉴얼](https://man7.org/linux/man-pages/man7/cgroups.7.html))

### (2) Namespace

![image](https://github.com/user-attachments/assets/27c86b43-c891-4bfc-ba2b-8e906c30b59a)

`프로세스가 어떤 자원을 볼 수 있는지 결정한다`

프로세스의 다양한 측면을 격리하는 Linux 커널 기능입니다.

한 프로세스 집합은 한 리소스 집합을 보고 다른 프로세스 집합은 다른 리소스 집합을 보도록 리소스를 분할하는 기능입니다. ([리눅스 메뉴얼](https://man7.org/linux/man-pages/man7/namespaces.7.html))

- 네임스페이스는 mount, uts, ipc, pid, neet, user까지 6가지 종류가 있습니다.
- `/proc/<pid>/ns` 경로로 직접 확인할 수 있습니다.
    
    ![image](https://github.com/user-attachments/assets/0468641b-9ff2-4965-a5b9-89c5486be482)

- `nsenter` 명령어로 특정 네임스페이스에 접속할 수 있습니다.
    
   ![image](https://github.com/user-attachments/assets/a54e29bf-5a97-428f-a847-ef0fa594e1bc)

위 6가지 네임스페이스 중, 2가지만 이야기해보겠습니다.

1. **마운트 네임스페이스**
    
    ![image](https://github.com/user-attachments/assets/cfec72e3-873a-4401-89fe-b6d8df2a7a7a)

    파일 시스템 마운트 포인트 격리하는 기능입니다.
    
    - 프로세스는 각자의 root 파일 시스템 (=`chroot` 와 유사)을 가질 수 있습니다.
    - 프로세스는 각자만의 Private 마운트를 가질 수도, 공유된 마운트를 가질 수 도 있습니다.
    
    ![image](https://github.com/user-attachments/assets/fce02803-989a-48ee-b3da-43c9159e5f9f)

    도커 컨테이너의 마운트는 커널의 mnt 네임스페이스 기능입니다.
    
    따라서 도커 컨테이너에 `exec -it bin/bash`  명령어로 접속한 뒤, ls 명령어를 실행한 결과는 namesapace로 접속한 마운트 경로와 실제로 동일합니다.
    
    ![image](https://github.com/user-attachments/assets/34df360c-8616-49aa-9ecd-9445bf14b043)

2. **PID 네임스페이스**
    
    PID (Process ID) Numberspace를 격리하는 기능입니다.
    
    ![image](https://github.com/user-attachments/assets/834b72dc-5ca1-42d1-aabb-004446bca50b)

    - 특정 PID 네임스페이스에 속한 프로세스는 같은 네임스페이스에 속한 프로세스만 볼 수 있습니다.
    - 각 PID 네임스페이스마다 1번 PID를 가집니다.
    - PID 1 프로세스가 종료되면, 전체 네임스페이스가 종료됩니다.
    - PID 네임스페이스는 중첩된 구조를 가질 수 있기에, 여러개의 PID 를 갖게 됩니다. (네임스페이스당 1개씩, 여러개)
        
    ![image](https://github.com/user-attachments/assets/bb1eceb4-40ff-4087-9201-29247a6353bd)
  
    1번 PID를 가진 프로세스는 특별한 프로세스입니다. pid 1은 `init 프로세스`로 커널이 생성합니다.
    
    ![image](https://github.com/user-attachments/assets/e8530598-7b32-47f1-a668-893cc0432b5b)

    다음은 1번 프로세스의 특징입니다.
    
    1. 시그널 처리 기능 (커널이 보내는 시그널을 자식 프로세스에게 전파)
    2. 좀비, 고아 프로세스 정리
    3. 1번 프로세스가 죽으면 시스템 패닉 -> reboot 으로만 해결 가능
    
    도커 프로세스는 자신의 PID 네임스페이스를 갖게 됩니다.
    
    따라서 컨테이너 내부로 접속해서 ps 명령어를 실행하면, 컨테이너 실행 태스크가 곧 PID 1번을 갖게 됩니다.
    
    ![image](https://github.com/user-attachments/assets/76364268-76de-4770-9055-a029fdc62710)

    우리가 컨테이너 Stop 명령어를 실행하면, 도커는 해당 PID 네임스페이스의 1번에게 종료 시그널을 보냅니다. 이로 인해 PID 1번이 종료되고, 해당 네임스페이스가 닫힙니다.
    
    ![image](https://github.com/user-attachments/assets/b882a0c4-5830-4178-b62b-d78b836daa8b)

    이때 주의할 점은, Init 프로세스와 다르게 컨테이너의 PID 1번은 자식 프로세스에게 종료 시그널을 전파하지 않는다는 점입니다. 따라서 컨테이너를 생성한 개발자가 직접 시그널을 전파하는 작업을 해줘야 합니다.
    
    ![image](https://github.com/user-attachments/assets/e0f57b40-0b9e-4553-8272-6d437b95f4f5)

    결국 도커는 리눅스의 기능을 활용하는 api 데몬이며, 내부적으로 리눅스 커널이 제공하는 `Cgroups`와 `Namespace` 기능을 이용해 컨테이너를 생성합니다.
    
    이번 글에서 도커가 내부적으로 어떻게 컨테이너를 생성하는지 간략히 소개해보았습니다.
    
    앞으로 프로젝트를 하며 도커를 사용함에 있어 작은 도움이 되셨길 바라며 글을 마치겠습니다.

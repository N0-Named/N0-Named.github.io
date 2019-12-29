---
layout: post
title: "Pwnable Setting"
date: 2019-12-29    
tags: [docker,pwnable,hacking,setting,linux,system]
comments: true
---

# Docker에서 Pwnable 환경 구축

저는 macOS를 사용하고 있어서 Pwnable을 할때 늘 VMware에 Ubuntu를 설치해서 사용해왔습니다. 하지만, 128GB를 사용하고 있어서 용량이 늘 부족합니다. 그래서 생각한 것이 docker를 이용해서 환경을 구축해보면 어떨까? 라는 생각을 하게 되었고, 찾아보니 이미 그렇게 사용하는 사람들이 많이 있었습니다.

우선 환경 구축에 앞서 기본적으로 [Docker](https://www.docker.com/)가 설치되어있어야 합니다.
[Docker](https://www.docker.com/) 설치법은 많은 블로거분들이 잘 설명해두었으니 설치방법은 생략하겠습니다.


## Docker Image 파일
처음 구축을 하기위해서 자료를 찾다가 유명하신 유튜버 LiveOverflow분이 Github에 올려둔 Image 파일을 발견하였습니다. 따로 Image를 만드는 번거로움을 줄이기 위해서 이 Image 파일을 이용해서 자신에 맞게 커스텀 해보겠습니다.

아래의 링크를 통해서 파일을 다운받아주세요.

[Image file Download](https://gist.github.com/LiveOverflow/b4502c5358a838d7ca9d92e8a2e8b5a0)

## Docker Build
터미널을 실행한 후 다운 받은 Image 파일이 있는 경로로 이동해주세요.
그리고 아래의 명령어를 통해서 Image 파일을 build합니다.
```
$ docker build -t ubuntu18:ctf - < Dockerfile
```

Image가 build되었는지 확인하기 위해서 아래의 명령어를 통해 확인합니다.

```
$ docker images

REPOSITORY     TAG     IMAGE ID              CREATED            SIZE

ubuntu18           ctf       6dde8c47ad56     2 minutes ago     1.47GB
```


## Docker Run
Build한 Image를 통해서 컨테이너를 실행할 수 있습니다.

```
$ docker run --rm -v $PWD:/pwd --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -p 5555:5555 -i ubuntu18:ctf
```

옵션은 아래와 같습니다.
```
# `--rm` : Clean up
# `-v` : Shared volume between the host(current directory) and the container(/pwd)
# `--cap-add & --security-opt` : Enable Ptrace and disable sandboxing for gdb
# `-p` : Port mapping for exposing # `-i` : Specify the docker image
```

그리고 새로운 터미널 창을 열어 아래의 명령어를 통해서 컨테이너를 사용할 수 있습니다.
```
$ docker ps
```

```
$ docker exec -it [컨테이너 이름 or ID] /bin/bash
# `exec` : Execute a program inside the container
# `-it` : Interactive and allocate a pseudo-TTY
root@1cc5488da0c2:/#  
```

## ETC
- 다양한 Pwnable 툴들이 설치되어 있고 docker를 실행할 때의 호스트 경로와 컨테이너의 /pwd 디렉터리를 통해서 파일 공유를 할 수 있어 정말 편합니다.
- gdb를 실행시켰을 때 기본으로 gef gdb가 실행됩니다. 저는 peda를 사용하기 때문에 **~/.gdbinit**의 설정을 변경해주고 사용하고 있습니다.
- 추가적으로 자신에게 편하게 커스텀해서 사용하면 정말 유용할 듯 합니다.

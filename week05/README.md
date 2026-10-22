# 5주차 - Docker
## Docker란
애플리케이션과 실행에 필요한 라이브러리, 설정 등을 컨테이너(Container)로 묶어서 서로 다른 환경에서도 동일하게 실행할 수 있도록 도와주는 플랫폼.
"내 컴퓨터에서는 되는데요?" 문제를 줄여주는 기술이다.

## 구성 요소
- Image : 컨테이너를 만들기 위한 읽기 전용 템플릿
- Container : 이미지로부터 생성된 인스턴스
- Dockerfile : 이미지를 어떻게 만들지 작성하는 파일 (다음 주차)
- Registry : 이미지를 저장 및 공유하는 곳 (Docker Hub, ghcr.io)

docker 명령어가 직접 컨테이너를 실행하는 것이 아니라 Docker Daemon(dockerd)에게 요청하고 dockerd가 실제 이미지와 컨테이너를 관리하는 Client-Server 구조로 되어있다.

## VM vs 컨테이너
VM은 하드웨어를 가상화하여 VM마다 Guest OS를 따로 실행하기 때문에 무겁고 시작이 느림. 대신 격리가 강하다.
컨테이너는 호스트 OS의 커널을 공유하고 실행 환경만 격리하기 때문에 가볍고 빠르게 시작된다.

격리는 두 가지로 이루어진다.
- namespace : 시야 격리. 컨테이너 안에서는 다른 프로세스, 네트워크, 파일 시스템이 보이지 않음
- cgroup : 자원 제한. 컨테이너가 사용할 수 있는 CPU, 메모리 상한을 정함

리눅스 컨테이너는 리눅스 커널이 필요하기 때문에 윈도우는 WSL2, 맥은 Linux VM 위에서 동작한다.

## 이미지 vs 컨테이너
이미지는 붕어빵 틀, 컨테이너는 붕어빵이다.
이미지 하나로 컨테이너를 여러 개 만들 수 있고 컨테이너 안에서 무엇을 바꿔도 이미지는 바뀌지 않는다.

# 실행 환경 확인
```
docker --version
docker run hello-world
```
```
Docker version 29.7.2, build a7dcaa6
```
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
```
로컬에 hello-world 이미지가 없었는데도 실행되었음. Docker Hub에서 이미지를 내려받아 컨테이너로 실행한 결과이다.

# nginx 컨테이너 실행
```
docker run -d -p 8080:80 --name web nginx
```
- -d : 백그라운드 실행 (안 붙이면 터미널이 묶임)
- -p 8080:80 : 호스트 포트 8080을 컨테이너 포트 80에 연결
- --name web : 컨테이너 이름 지정
- nginx : 사용할 이미지 이름

호스트에 nginx를 설치하지 않았지만 nginx 이미지를 받아 컨테이너로 실행했기 때문에 서버가 뜬다.

## curl -I http://localhost:8080 결과
```
HTTP/1.1 200 OK
Server: nginx/1.31.8
Date: Thu, 22 Oct 2026 04:49:15 GMT
Content-Type: text/html
Content-Length: 896
Last-Modified: Wed, 21 Oct 2026 14:24:51 GMT
Connection: keep-alive
ETag: "6ad8cb33-380"
Accept-Ranges: bytes
```

## docker ps 결과
```
CONTAINER ID   IMAGE     COMMAND                   CREATED        STATUS                  PORTS                                     NAMES
40ab0c762cd9   nginx     "/docker-entrypoint.…"   1 second ago   Up Less than a second   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web
```

## 여러 컨테이너 실행
```
docker run -d -p 8081:80 --name web2 nginx
docker run -d -p 8082:80 --name web3 nginx
```
컨테이너 안의 포트는 모두 80이지만 각 컨테이너가 자기만의 네트워크 공간을 가지기 때문에 충돌하지 않음.
단, 호스트 포트(8080, 8081, 8082)와 컨테이너 이름은 서로 달라야 한다.

```
$ docker run -d -p 8080:80 --name web4 nginx
docker: Error response from daemon: ... Bind for 0.0.0.0:8080 failed: port is already allocated

$ docker run -d -p 8084:80 --name web nginx
docker: Error response from daemon: Conflict. The container name "/web" is already in use by container ...
```
첫 번째는 호스트 포트 충돌, 두 번째는 컨테이너 이름 충돌이다.
포트 충돌의 경우 컨테이너 자체는 생성되어 있기 때문에 `docker rm -f web4`로 정리해줘야 한다.

# 도커 명령어
```
docker ps                    # 실행 중인 컨테이너 출력
docker ps -a                 # 멈춘 컨테이너까지 전체 출력
docker images                # 내려받은 이미지 목록
docker logs web              # 현재까지의 로그 확인
docker logs -f web           # 로그 실시간 확인 (종료는 Ctrl + C)
docker stop web2 web3        # 정지
docker start web2            # 다시 시작
docker restart web3          # stop 후 start
docker rm -f web2 web3       # 삭제 (-f : 실행 중이어도 강제 삭제)
```
stop 한 컨테이너는 `docker ps`에서는 사라지지만 `docker ps -a`에서 Exited 상태로 남아있다.
rm으로 컨테이너를 삭제해도 nginx 이미지는 로컬에 그대로 남아있음.

# 컨테이너 안으로
```
docker exec -it web bash
```
- -i : stdin을 열어 키보드 입력을 받을 수 있게 함
- -t : 가상 터미널(TTY)을 할당

프롬프트가 `root@컨테이너ID:/#` 형태로 바뀌면 컨테이너 안에 들어온 것이다.
`cat /etc/os-release`로 확인해보면 nginx 이미지는 Debian 기반으로 되어있음.

exit로 나가도 컨테이너는 살아있다. exec로 띄운 bash는 실행 중인 컨테이너에 추가로 실행한 프로세스이기 때문.

## 컨테이너 내부 파일 수정 후 삭제
```
root@ed35db34e032:/# echo "<h1>Hi</h1>" > /usr/share/nginx/html/index.html
```
수정 후 접속하면 Hi가 출력되지만 컨테이너를 삭제하고 같은 이미지로 다시 실행하면 원래의 Welcome to nginx! 페이지로 돌아온다.
변경 내용은 컨테이너의 쓰기 계층에 저장되기 때문에 컨테이너를 삭제하면 함께 사라짐. 틀(이미지)은 바뀌지 않는다.

# Docker 활용
## 환경 변수 주입 (-e)
```
$ docker run -d -p 8083:80 -e MY_NAME=Kam --name env_test nginx
$ docker exec env_test env | grep MY_NAME
MY_NAME=Kam
```

## 일회용 컨테이너 (--rm)
--rm 옵션을 붙이면 실행이 끝난 후 컨테이너가 자동으로 삭제된다.
```
$ docker run --rm ubuntu:24.04 echo "hi"
hi
$ docker run --rm python:3.12-slim python -c "print(0.1+0.2)"
0.30000000000000004
```
`docker run -it --rm ubuntu:24.04 bash`로 실행하면 호스트에 상관없이 Ubuntu 환경을 바로 써볼 수 있다.

## 서로 다른 파이썬 버전 실행
호스트에 파이썬 3.9, 3.12를 설치하지 않아도 테스트가 가능하다.
```
$ docker run --rm python:3.9-slim python --version
Python 3.9.25
$ docker run --rm python:3.12-slim python --version
Python 3.12.15
```

파이썬 3.10부터 지원되는 match-case 문법으로 test.py를 작성하여 비교하였다.
```
$ docker run --rm -v "$PWD:/app" -w /app python:3.9-slim python test.py
  File "/app/test.py", line 2
    match value:
          ^
SyntaxError: invalid syntax

$ docker run --rm -v "$PWD:/app" -w /app python:3.12-slim python test.py
two
```
- -v "$PWD:/app" : 현재 디렉터리를 컨테이너의 /app에 연결
- -w /app : 컨테이너의 작업 디렉터리를 /app으로 지정

3.9에서는 match 문법을 몰라서 SyntaxError가 발생하고 3.12에서는 정상 실행된다.

# 혼자서 해보기
## nginx1, nginx2, nginx3 실행
```
docker run -d -p 8091:80 --name nginx1 nginx
docker run -d -p 8092:80 --name nginx2 nginx
docker run -d -p 8093:80 --name nginx3 nginx
```

각 컨테이너에 exec로 들어가서 index.html을 수정하였다.
```
docker exec -it nginx1 bash
root@aa4c290c3b85:/# echo "<h1>Welcome to nginx1</h1>" > /usr/share/nginx/html/index.html
root@aa4c290c3b85:/# exit
```
nginx2, nginx3도 동일하게 진행.

## curl 결과
```
$ curl http://localhost:8091
<h1>Welcome to nginx1</h1>

$ curl http://localhost:8092
<h1>Welcome to nginx2</h1>

$ curl http://localhost:8093
<h1>Welcome to nginx3</h1>
```

## docker ps 결과
```
CONTAINER ID   IMAGE     COMMAND                   CREATED          STATUS          PORTS                                     NAMES
3fda208863a8   nginx     "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:8093->80/tcp, [::]:8093->80/tcp   nginx3
ddf407f94b36   nginx     "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:8092->80/tcp, [::]:8092->80/tcp   nginx2
aa4c290c3b85   nginx     "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:8091->80/tcp, [::]:8091->80/tcp   nginx1
c43849b1d511   nginx     "/docker-entrypoint.…"   53 seconds ago   Up 52 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web
```

## 정리
```
docker rm -f nginx1 nginx2 nginx3 web
```

# GitHub Issue
Issue는 오류만 적는 곳이 아니라 무엇을, 왜 해야 하는지 기록하고 진행 상황을 관리하는 곳이다.
이번 실습도 완료 조건이 있는 하나의 작업으로 보고 Issue로 관리하였다.

Issue → Branch → Pull Request → Merge 순서로 진행한다.

## 브랜치 생성
```
git switch main
git pull --ff-only
ISSUE_NO=6
git switch -c "docs/issue-${ISSUE_NO}-docker-start"
```
이슈 번호를 변수에 저장해두면 브랜치 이름, 커밋 메시지, PR 본문에 그대로 사용할 수 있다.

main에 커밋하지 않은 변경 사항이 있는데 새 브랜치로 가져가고 싶지 않다면 `git stash`로 임시 보관해둔다.
`git stash pop`은 다시 적용하면서 stash 목록에서 제거하고 `git stash apply`는 목록에 그대로 남겨둔다.
새로 만든 파일(untracked)은 기본적으로 포함되지 않아서 `git stash -u`를 써야 함.

## 커밋, 푸시, PR
```
git add ~/devops/week05/
git commit -m "5주차: Docker 실행환경 및 nginx 확인 (#${ISSUE_NO})"
git push -u origin "docs/issue-${ISSUE_NO}-docker-start"
gh pr create --title "5주차: Docker" --body "Close #${ISSUE_NO} ..."
```
PR 본문에 `Close #이슈번호`가 있으면 Merge 될 때 이슈가 자동으로 종료된다.
close 외에도 closes, closed, fix, fixes, fixed, resolve, resolves, resolved 사용 가능.

## Merge 후 확인
```
gh pr merge --squash --delete-branch
git switch main
git pull --ff-only
gh issue view "$ISSUE_NO" --json state -q .state
```
결과가 CLOSED로 나오면 이슈가 정상적으로 종료된 것이다.

# 2주차 - 실습 환경 구축과 버전 관리 기초
## 서버
서버는 하드웨어이기도 하고 소프트웨어이기도 함. "전원을 끈다"면 하드웨어, "서버를 설치했다"면 소프트웨어로서의 서버이다.
Serve(제공하다)에서 나온 말로 무언가를 제공하는 주체를 뜻한다.
클라이언트가 원하는 것을 요청(Request)하면 서버는 그 요청에 대한 응답(Response)을 돌려준다.

## 서버 운영 방법
- 베어메탈 : PC 한 대에 OS를 설치하고 여러 소프트웨어를 운영
- 하이퍼바이저 : PC 한 대에 여러 가상 머신을 운영
- 컨테이너 : PC 한 대에 여러 컨테이너를 운영

베어메탈은 하나의 OS에서 모든 프로그램을 관리하기 쉽지만 프로그램 하나가 해킹당하거나 리소스를 많이 쓰면 나머지 프로그램도 위험해짐.
이 단점을 해결하기 위해 가상화 기술로 격리된 공간에서 소프트웨어를 실행한다.

그렇다고 하드웨어를 여러 대 쓰는 것은 비용, 설치 공간, 인력 등을 생각하면 비경제적임. 높은 사양의 컴퓨터 한 대를 가상화해서 쓰는 편이 낫다.

## 가상화
물리적인 컴퓨팅 환경 안에서 여러 개의 논리적인 컴퓨팅 환경을 만드는 기술.
실제 서버가 없어도 마치 있는 것처럼 만들 수 있다.

### 하이퍼바이저
호스트 OS의 리소스(CPU, 메모리, 디스크, 네트워크)를 할당하여 게스트 OS(가상 머신)를 실행한다.
각 가상 머신이 자신의 게스트 OS와 커널을 따로 가지기 때문에 오버헤드가 크다.

```
프로그램 → 시스템 콜 → 게스트 OS 커널 → 가상 하드웨어 → 하이퍼바이저 → 실제 하드웨어
```

커널은 프로그램의 요청을 받아 하드웨어 리소스를 안전하게 관리하고 연결해주는 역할을 하며 이를 위한 시스템 콜(system call)을 제공한다.

### 컨테이너
별도의 게스트 OS 커널 없이 호스트 OS의 커널 기능으로 격리된 공간을 만든다. 그래서 하이퍼바이저보다 가볍고 빠름.
모든 컨테이너가 호스트 OS의 커널을 공유하기 때문에 호스트 OS와 다른 종류의 OS는 실행할 수 없다.

- 네임스페이스(namespace) : 프로세스, 파일 시스템, 네트워크 등의 격리
- 제어 그룹(cgroups) : 리소스 사용량 제어

# WSL
Windows Subsystem for Linux. 윈도우에서 리눅스를 사용할 수 있게 해준다.
가상 머신보다 리소스를 적게 쓰고 윈도우 파일 탐색기에서도 쉽게 접근할 수 있으며 bash 명령어를 그대로 쓸 수 있음.

설치 전에 작업 관리자(Ctrl + Shift + Esc) > 성능 > CPU에서 가상화가 사용으로 되어있는지 확인해야 한다.
사용 안 함이면 BIOS에서 Virtualization을 Enabled로 바꿔줘야 함.

## 설치
파워셸을 관리자 권한으로 실행 후 진행한다.
```
wsl -l -v
wsl --install
```
- -l : list, 목록
- -v : verbose, 상세 정보

`wsl --install`은 WSL 기능 활성화, 가상 머신 플랫폼 활성화, 최신 리눅스 커널 설치, 기본 Ubuntu 설치까지 한 번에 해준다.

```
wsl --list --online                    # 설치 가능한 배포판 목록
wsl --install -d Ubuntu-24.04          # 특정 배포판 설치 (-d : distribution)
wsl --unregister Ubuntu                # 제거
```

## wsl -l -v 결과
```
  NAME              STATE           VERSION
* Ubuntu            Running         2
  Ubuntu-24.04      Stopped         2
  docker-desktop    Running         2
```
- STATE : 현재 실행 중인지
- VERSION : WSL 1인지 2인지
- \* : wsl만 입력했을 때 실행될 기본 배포판

## 버전 설정
```
wsl --set-default-version 2            # 앞으로 설치할 배포판의 기본값을 2로
wsl --set-version Ubuntu-24.04 2       # 이미 설치된 특정 배포판을 2로 변경
wsl --set-default Ubuntu-24.04         # 기본으로 실행할 배포판 설정
```
세 명령어가 비슷하게 생겨서 헷갈리기 쉬움.

## 실행
```
wsl
wsl -d Ubuntu-24.04
exit
```
exit를 입력하면 파워셸로 돌아온다.

학교 PC처럼 여러 명이 쓰는 경우에는 `sudo adduser <계정이름>`으로 본인 계정을 만들고 `wsl -d Ubuntu-24.04 -u <계정이름>`으로 접속한다.
sudo는 superuser do의 약자로 root 권한으로 명령어를 실행함. 시스템 전체에 영향을 줄 수 있으니 조심해서 써야 한다.

## Ubuntu 사용 준비
```
mkdir ~/devops
cd ~/devops
sudo apt update
sudo apt upgrade -y
```
`~`는 리눅스 전용 공간인 홈 디렉터리이다.
apt(advanced package tool)는 패키지 관리 도구이며 `sudo apt install <패키지>`, `sudo apt remove <패키지>`로 설치, 제거한다.
-y는 중간에 나오는 yes/no 질문에 자동으로 yes를 입력해준다.

# 기본 리눅스 명령어
## 디렉터리
```
pwd                 # 현재 위치 확인 (print working directory)
ls                  # 파일/폴더 목록 (list)
ls -l               # 권한, 소유자, 크기, 수정 시간 등 상세 출력
ls -a               # 숨김 파일까지 출력 (.으로 시작하는 파일)
cd ~                # 홈 디렉터리로
cd ..               # 상위 디렉터리로
cd -                # 직전에 작업하던 디렉터리로
mkdir <디렉터리>    # 생성
rmdir <디렉터리>    # 삭제 (비어있을 때만 가능)
```

## 권한
```
$ ls -l s.txt
-rw-r--r-- 1 sikgong sikgong 19 Oct 22 13:59 s.txt
```
맨 앞 10자리 중 첫 글자는 파일 종류(- 파일, d 디렉터리, l 심볼릭 링크)이고 나머지 9자리는 3자리씩 소유자, 그룹, 기타 사용자의 권한이다.
r은 read, w는 write, x는 execute.

권한은 chmod(change mode)로 바꾼다. 숫자로는 r=4, w=2, x=1을 더해서 표기함.
```
$ chmod 755 s.txt
$ ls -l s.txt
-rwxr-xr-x 1 sikgong sikgong 19 Oct 22 13:59 s.txt

$ chmod g-w,o-x s.txt
$ ls -l s.txt
-rwxr-xr-- 1 sikgong sikgong 19 Oct 22 13:59 s.txt
```
755는 소유자 7(4+2+1) 읽기/쓰기/실행, 그룹 5(4+1) 읽기/실행, 기타 5(4+1) 읽기/실행이다.

## 파일 다루기
```
$ mkdir test
$ cd test
$ touch devops.txt
$ echo "Hello devops" > devops.txt
$ cat devops.txt
Hello devops
```
touch는 빈 파일을 만들고 cat(concatenate)은 파일 내용을 출력한다.

```
$ echo "Hello, " > 1.txt
$ echo "DevOps" > 2.txt
$ cat 1.txt 2.txt > s.txt
$ cat s.txt
Hello, 
DevOps

$ echo "Hi!" > 3.txt
$ cat 3.txt >> s.txt
$ cat s.txt
Hello, 
DevOps
Hi!
```
`>`는 파일을 새로 덮어쓰고 `>>`는 파일 뒤에 이어서 추가한다.

```
cp devops.txt hi.txt      # 복사
mv hi.txt hello.txt       # 이동 (같은 디렉터리면 이름 변경과 같음)
rm s.txt                  # 삭제
```

디렉터리 안에 파일이 남아있으면 rmdir로 삭제가 안 됨.
```
$ rmdir test
rmdir: failed to remove 'test': Directory not empty
$ rm test/devops.txt
$ rmdir test
```
`rm -rf`는 안에 있는 것까지 전부 지워버리기 때문에 함부로 쓰면 안 된다.

## 기타 정보 확인
```
$ whoami
sikgong

$ uname -a
Linux KSH 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64 GNU/Linux

$ cat /etc/os-release | head -4
PRETTY_NAME="Ubuntu 26.04 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04 (Resolute Raccoon)"

$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       1.2Gi       5.9Gi        69Mi       786Mi       6.4Gi
Swap:          2.0Gi          0B       2.0Gi
```
- who : 지금 로그인 중인 사용자
- id : UID, GID, 그룹 정보
- df -h : 디스크 사용량 (h : 사람이 읽기 쉽게)
- free -h : 메모리 상태. Swap은 메모리가 부족할 때 일부 데이터를 디스크로 옮겨 쓰는 영역이다.
- clear : 화면 정리

## history
```
history          # 지금까지 입력한 명령어
!103             # 103번째 명령어 실행
!103:p           # 103번째 명령어 출력만
!!               # 바로 직전 명령어 다시 실행
!ls              # 가장 최근 ls로 시작한 명령어 다시 실행
sudo !!          # 직전 명령어를 sudo로 다시 실행
```
`apt update`를 sudo 없이 실행해서 권한 오류가 났을 때 `sudo !!`로 바로 다시 실행할 수 있어서 편함.

## 파일 내용 보기
```
cat <파일>          # 전체 출력
less <파일>         # 긴 파일 보기
head <파일>         # 앞부분만
tail <파일>         # 뒷부분만
tail -f <로그파일>  # 실시간으로 계속 보기 (종료 : Ctrl + C)
```

# Git
코드의 변경 이력을 기록하고 관리하는 버전 관리 시스템.
이전 버전으로 돌아가거나 변경 내용을 비교할 수 있고 브랜치를 만들어 독립적으로 작업할 수 있다. 인터넷 없이도 로컬에서 사용 가능함.

## 설치 및 설정
```
sudo apt install git -y
git --version
git config --global user.name "이름"
git config --global user.email "이메일"
git config --global init.defaultBranch main
git config --list
```
```
git version 2.53.0
```

## 저장소 만들기
```
cd ~/devops
git init
ls -a
git status
```
git init을 하면 숨겨진 .git 디렉터리가 생기고 여기에 커밋 기록, 브랜치 정보, 설정 등이 저장된다.
홈 디렉터리 전체가 아니라 프로젝트 디렉터리(~/devops)에서 해야 하기 때문에 위치를 꼭 확인해야 함.

## 커밋
```
cd ~/devops/week02
echo "Hello, DevOps!" > text.txt
git status
git add text.txt
git status
git commit -m "first commit"
git log
```
git add는 파일을 Git에 저장하는 것이 아니라 다음 커밋에 포함할 변경 사항을 고르는 작업이다. 즉 staging area에 넣는 것.
git commit은 준비된 변경 사항을 하나의 버전(스냅샷)으로 기록한다. 게임의 세이브 포인트와 비슷함.
`git add .`은 수정한 모든 파일을 지정한다.

## 버전 이동과 비교
```
echo "Inha tech college" >> text.txt
git add .
git commit -m "Second Commit"
```
```
$ git log --oneline
57158ae Second Commit
1f3d3cd first commit
```

커밋 해시로 이전 버전으로 이동할 수 있다. 커밋을 구분할 수 있을 만큼 앞부분만 입력해도 됨.
```
$ git switch -d 1f3d3cd
$ cat text.txt
Hello, DevOps!
```
-d는 detach. 예전에는 `git checkout <커밋해시>`를 사용했다.

```
$ git diff 1f3d3cd 57158ae
diff --git a/week02/text.txt b/week02/text.txt
index 02ee7eb..59b7838 100644
--- a/week02/text.txt
+++ b/week02/text.txt
@@ -1 +1,2 @@
 Hello, DevOps!
+Inha tech college
```
+는 두 번째 커밋에 새로 생긴 부분, -는 삭제된 부분이다.

## 브랜치
특정 커밋을 가리키는 이름. 서로 다른 작업을 독립적으로 진행할 수 있다.
```
echo "hello" > hello.txt
git add .
git commit -m "hello"
git switch -c hi
echo "hi" > hello.txt
git add .
git commit -m "hello -> hi"
git switch main
cat hello.txt        # hello
git switch hi
cat hello.txt        # hi
```
같은 파일인데 브랜치에 따라 내용이 다르게 나옴.

```
$ git diff main hi
diff --git a/week02/hello.txt b/week02/hello.txt
index ce01362..45b983b 100644
--- a/week02/hello.txt
+++ b/week02/hello.txt
@@ -1 +1 @@
-hello
+hi
```

```
git switch -c <브랜치>              # 브랜치 생성 후 이동 (-c : create)
git switch <브랜치>                 # 이동
git branch                          # 현재 브랜치 확인
git log --oneline --all --graph     # 브랜치 포함 그래프로 보기
```
```
| * af1d77a hello -> hi
|/  
* f4d5222 hello
* 57158ae Second Commit
* 1f3d3cd first commit
```

# GitHub
Git 저장소를 온라인에 저장하고 공유하는 협업 플랫폼.
Pull Request, Issue, Code Review 같은 협업 기능과 GitHub Actions 같은 CI/CD 기능을 제공한다.

## gh로 연결
```
sudo apt install gh -y
gh --version
gh auth login
```
```
gh version 2.46.0 (2025-12-13 Ubuntu 2.46.0-4)
```
gh auth login 후 GitHub.com, HTTPS, Login with a web browser 순서로 선택하고 브라우저에서 one-time code를 입력하면 연결된다.
브라우저가 안 열리면 https://github.com/login/device 로 접속하거나 `sudo apt install wslu -y`를 설치한다.

## 저장소 만들고 올리기
```
gh repo create devops --public --source=. --remote=origin --push
```
- --public : 공개 저장소로 생성
- --source=. : 현재 디렉터리의 Git 저장소를 GitHub에 연결
- --remote=origin : 원격 저장소 이름을 origin으로 지정 (관례적으로 사용하는 이름)
- --push : 로컬의 커밋을 원격 저장소에 업로드

이후에는 커밋 후 반드시 `git push`로 올려야 한다.
```
gh repo view --web          # 웹에서 보기
gh browse --no-browser      # 링크만 보기
```


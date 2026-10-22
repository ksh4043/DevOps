# 3주차 - 셸 스크립팅
## 셸 스크립트를 쓰는 이유
로그에서 오류 개수 세기, 백업 폴더로 복사, 날짜 붙이기, 오래된 백업 삭제 같은 작업을 매일 손으로 하면 하루 10분이라도 1년이면 약 61시간이 된다.
게다가 사람은 오타, 순서 바뀜, 생략 같은 실수를 반드시 함.
스크립트로 작성해두면 기계는 몇 번을 시켜도 똑같이 실행한다. (단, 제대로 작성된 스크립트인 경우)

# Qwen w/ Ollama
- Qwen : 알리바바 클라우드에서 개발한 대형 언어 모델. 이번 실습에서는 qwen3:0.6b 사용
- Ollama : 로컬 환경에서 LLM을 쉽게 설치하고 실행할 수 있게 해주는 오픈소스 플랫폼

작은 모델이라 한국어 답변이 어색하거나 틀릴 수 있음. 답변 실력보다는 실행, 종료, 재실행 흐름을 확인하는 것이 목적이다.

## 구조
```
브라우저 (http://localhost:8000)
    ↓ 질문 전송 / 답변 표시
Python 웹앱 (chat.py, 127.0.0.1:8000)
    ↓ 질문을 API로 전달
Ollama 서버 (127.0.0.1:11434)
    ↓ 모델 실행
qwen3:0.6b
```
chat.py는 파이썬 표준 라이브러리만 사용하기 때문에 Flask 같은 걸 따로 설치할 필요가 없다.
모델은 Ollama가 따로 보관하기 때문에 이 폴더에 복사하거나 GitHub에 올리면 안 됨.

## 설치
파워셸이 아닌 Ubuntu 터미널에서 실행한다.
```
sudo apt update
sudo apt install -y python3 curl zstd
curl -fsSL https://ollama.com/install.sh | sh
```

## 서버 확인
```
$ ollama --version
ollama version is 0.34.1

$ python3 --version
Python 3.14.4

$ curl -fsS http://127.0.0.1:11434/api/tags
{"models":[{"name":"qwen3:0.6b","model":"qwen3:0.6b", ... }]}
```
JSON 형태로 뭔가 나오면 Ollama 서버가 응답한 것이다. 연결이 안 되면 `ollama serve`로 서버를 직접 띄운다.

## 모델 다운로드
```
$ ollama pull qwen3:0.6b
$ ollama list
NAME          ID              SIZE      MODIFIED    
qwen3:0.6b    7df6b6e09427    522 MB    5 weeks ago    
```
b는 Billion(10억)으로 모델의 파라미터 개수이다. 0.6b는 약 6억 개.
파라미터가 클수록 성능이 좋아지는 경향이 있지만 그만큼 느리고 저장 공간, 메모리가 더 필요함.

## 사전 응답 점검
```
curl --max-time 90 -fsS http://127.0.0.1:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen3:0.6b", "prompt":"안녕! 한 문장으로 인사해봐!", "think":false, "stream":false, "options":{"num_ctx":8192,"num_predict":512}}'
```
```
{"model":"qwen3:0.6b","created_at":"2026-10-22T04:59:59.132164587Z","response":"안녕하세요!","done":true,"done_reason":"stop", ... }
```
- stream: false : 응답을 한 번에 보여줌 (기본은 스트리밍)
- think: false : 생각 모드 비활성화

결과는 매번 달라질 수 있다.

# 웹 브라우저에서 열기
```
cd ~/devops
mkdir -p week03/qwen-web
cd week03/qwen-web
cp "/mnt/c/Users/PC/Downloads/chat.py" .
python3 chat.py
```
-p는 parent로 필요한 상위 디렉터리까지 함께 만들어준다.
`explorer.exe .`로 탐색기를 열어서 chat.py를 드래그해 넣어도 됨.

```
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
```

Ctrl + C로 종료하면 Python 웹앱이 꺼진 것이고 Ollama 서버는 그대로 켜져있다.
그런데 며칠 지나서 다시 실행하려면 `cd ~ → cd devops → cd week03 → cd qwen-web → python3 chat.py` 경로를 다 외워야 함. 그래서 셸 스크립트로 만든다.

# 셸 스크립트
## start.sh
```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
exec python3 chat.py
```

### #!/bin/bash
셔뱅(shebang). 어떤 인터프리터로 실행할지 OS에 알려주는 부분으로 파일의 첫 줄에 작성한다.
`#` 하나만 있으면 주석으로 처리됨.

파이썬으로도 확인해볼 수 있다. run_py.sh
```python
#!/usr/bin/env python3

print('Hello, DevOps!')
```
```
$ chmod +x run_py.sh
$ ./run_py.sh
Hello, DevOps!
```
`/usr/bin/env python3`는 PATH에서 python3를 찾아서 실행한다.

### cd "$(dirname "$0")" || exit 1
스크립트가 있는 디렉터리로 이동하고, 이동에 실패하면 스크립트를 종료한다.
- $0 : 현재 실행한 스크립트의 이름 또는 경로
- dirname : 경로에서 파일 이름을 빼고 디렉터리 부분만 출력
- $( ... ) : 명령어 치환. 괄호 안의 명령어 실행 결과를 그 자리에 넣음
- "$0" : 경로에 공백이 있을 수 있어서 따옴표 사용

```
$ echo "$(pwd)"
/home/sikgong/DevOps/week03/qwen-web
```
`pwd`를 실행한 것과 같은 결과가 나온다.

이 줄 덕분에 다른 위치에서 실행해도 chat.py를 찾을 수 있음.
```
$ cd ~
$ ./DevOps/week03/qwen-web/start.sh
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
```

[참고] 그 외 특수 변수
- $1, $2 : 첫 번째, 두 번째 인자
- $# : 전달받은 인자 개수
- $@ : 전달받은 모든 인자
- $? : 직전 실행한 명령의 종료 상태

### || exit 1
리눅스 명령어는 성공 또는 실패 값을 남긴다. 성공은 0, 실패는 0 이외의 값.
`||`는 앞 명령이 실패하면 뒤 명령을 실행하는 논리 OR이다.
숫자 0은 거짓이지만 종료 상태 0은 성공을 뜻하기 때문에 헷갈리면 안 됨.
`exit 1`은 실패를 의미하는 상태 코드 1을 남기고 종료한다.

### exec python3 chat.py
현재 bash를 python3로 대체한다. (PID 유지)
exec 뒤에 `echo "End"`를 붙이고 Ctrl + C로 종료해서 비교해봤다.

```
$ ./exec_end.sh          # exec python3 chat.py 뒤에 echo "End"
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
웹앱을 종료했습니다.

$ ./noexec_end.sh        # python3 chat.py 뒤에 echo "End"
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
웹앱을 종료했습니다.
End
```
exec를 쓰면 bash가 python3로 바뀌어버려서 그 뒤의 줄은 실행되지 않음.

## 실행
```
$ ls -l start.sh
-rw-r--r-- 1 sikgong sikgong 169 Oct 22 13:59 start.sh

$ start.sh
start.sh: command not found

$ ./start.sh
./start.sh: Permission denied

$ chmod u+x start.sh
$ ls -l start.sh
-rwxr--r-- 1 sikgong sikgong 169 Oct 22 13:59 start.sh
```
- Permission denied : 실행 권한이 없어서 발생. `chmod u+x`로 권한을 준다.
- command not found : 명령어 이름만 쓰면 PATH에 등록된 디렉터리에서 찾기 때문에 `./start.sh`로 실행해야 함.

`export PATH="$PATH:$HOME/devops/week03/qwen-web/"`로 PATH에 추가하면 이름만으로도 실행되지만 재접속하면 해제된다.

```
$ ./start.sh
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
```
```
질문 : 강아지 이름 세 개만 추천해줄래?
답변 : 강아지 이름 추천해드릴게요. 🐾
```
0.6b 모델이라 이름을 추천해준다고만 하고 실제로는 추천을 안 해줌.

# 환경 변수
chat.py는 MODEL, WEB_PORT 환경 변수를 읽어서 사용 모델과 포트를 정한다.

## start_with_export.sh
```
$ cat > start_with_export.sh << 'EOF'
> #!/bin/bash
> cd "$(dirname "$0")" || exit 1
> export MODEL="qwen3:0.6b"
> exec ./start.sh
> EOF
$ chmod u+x start_with_export.sh
$ ./start_with_export.sh
웹 접속: http://localhost:8000
사용 모델: qwen3:0.6b
종료: Ctrl+C
```
스크립트 안에서 export한 값은 원래 터미널의 환경 변수를 바꾸지 않는다.

## heredoc
`cat > 파일 << 'EOF'`는 입력하는 여러 줄의 내용을 파일에 저장하다가 EOF 줄이 나오면 입력을 끝낸다.
EOF 대신 END, FINISH 등 다른 문자열을 써도 됨.

EOF에 따옴표를 붙이면 변수가 문자 그대로 저장되고 안 붙이면 현재 값으로 바뀌어서 저장된다.
```
$ MODEL="qwen3"
$ cat > test.sh << 'END'
> echo "$MODEL"
> END
$ cat test.sh
echo "$MODEL"

$ cat > test.sh << END
> echo "$MODEL"
> END
$ cat test.sh
echo "qwen3"
```

## 실습 - 포트 8080으로 실행
start_with_export_2.sh
```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
export WEB_PORT="8080"
exec ./start.sh
```
```
$ ./start_with_export_2.sh
웹 접속: http://localhost:8080
사용 모델: qwen3:0.6b
종료: Ctrl+C
```
```
질문 : 학교 갈 때 필요한 물건 세 개만 말해줘
답변 : 학교 갈 때 필요한 물건 세 개는 택시, 티켓, 지도.
```
http://localhost:8080 으로 접속이 된다. 답변은 역시 이상함.

# 조건문
python3가 없을 때 알려주는 부분을 start.sh에 추가하였다.
```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
if ! command -v python3 >/dev/null 2>&1; then
	echo "python3를 먼저 설치하세요." >&2
	exit 1
fi
exec python3 chat.py
```

## command -v
명령어가 있으면 경로를 출력하고 없으면 아무것도 출력하지 않고 실패 상태를 반환한다.
```
$ command -v python3
/usr/bin/python3
$ echo $?
0

$ command -v python4
$ echo $?
1
```
`!`를 붙이면 "찾을 수 없다면"이 된다.

## >/dev/null 2>&1
- 0 : 표준 입력(stdin), 1 : 표준 출력(stdout), 2 : 표준 오류(stderr)
- `>/dev/null` : 표준 출력을 /dev/null(쓰레기통)로 보내서 버림
- `2>&1` : 표준 오류를 표준 출력이 가는 곳으로 보냄

알고 싶은 건 있는지 없는지뿐이라 출력은 전부 버린다.
순서가 바뀌어서 `2>&1 >/dev/null`이 되면 표준 오류는 원래 표준 출력이 향하던 화면에 남기 때문에 순서가 중요함.

## >&2 와 exit 1
`echo "..." >&2`는 메시지를 일반 출력이 아닌 오류 메시지로 출력한다.
오류 메시지로 보내는 것과 오류 상태로 종료하는 것은 다르기 때문에 `exit 1`을 따로 써줘야 실패로 처리됨.

[참고] `uname`이 Darwin이면 맥, 아니면 Ubuntu로 구분해서 OS별 설치 명령어를 안내하도록 더 친절하게 만들 수도 있다.

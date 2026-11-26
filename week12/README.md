# week12: 깃허브 액션(CI)

## 구성
- `.github/workflows/build.yml`: 린트 → 테스트 → 이미지 빌드 → (main일 때만) ghcr.io 푸시
- `guestbook/tests/test_app.py`: 앱 동작 테스트 코드
- 트리거: main 푸시(`guestbook/**`, 워크플로 변경 시), main 대상 PR, 수동 실행(workflow_dispatch)
- 이미지 태그: 커밋 해시 7자리 + (기본 브랜치일 때) `latest`

## 테스트 내용

| 단계 | 명령 | 확인하는 것 |
|---|---|---|
| 코드 검사(Lint) | `ruff check . --select=E9,F` | E9: 문법 오류 / F: 정의 안 된 변수, 안 쓰는 import, 오타 |
| 앱 동작 테스트 | `python -m unittest discover -s tests -v` | 아래 두 테스트 |

- `test_health`: `GET /healthz` 응답 코드가 200이고 `status`가 `ok`인지 확인
- `test_add_and_read`: `POST /add`로 글 하나 등록 → 302(redirect) 확인 → `GET /api/entries`에서 글이 1개이고 내용이 `hello`인지 확인
- `setUp`에서 메모리 저장소를 비워 테스트끼리 영향을 주지 않게 함 (`REDIS_HOST` 없음 → 메모리 모드)

## 결과 (푸시 전 컨테이너로 확인)

```
$ docker run --rm -v "$PWD":/app -w /app -e PYTHONDONTWRITEBYTECODE=1 python:3.12-slim \
  sh -c "pip install -q ruff -r requirements.txt && ruff check . --select=E9,F --no-cache \
  && python -m unittest discover -s tests -v"
All checks passed!
[INFO] REDIS_HOST 없음 → 메모리 모드로 동작합니다.
test_add_and_read (test_app.GuestbookTest.test_add_and_read) ... ok
test_health (test_app.GuestbookTest.test_health) ... ok
----------------------------------------------------------------------
Ran 2 tests in 0.023s

OK
```

- 린트: 통과 (종료 코드 0)
- 테스트: 2개 모두 통과
- PR에서는 검사와 빌드만 하고, main에 머지된 뒤에 ghcr.io로 이미지를 푸시함

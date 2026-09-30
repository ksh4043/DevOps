# 4주차 학습 내용 요약(GitHub 관련)
## branch 분리
main branch에 모든 팀원이 작업한 산출물을 업로드/합치는 행위는 프로젝트 관리 차원에서 바람직하지 못 함.

```
git switch -c [카테고리/기능명]
```

switch 명령어로 브랜치를 변경하여 작업한다.
-c 는 새로운 브랜치를 생성하고 현재 브랜치를 생성한 브랜치로 옮긴다.
카테고리는 [feat, fix, docs 등]이 있으며 순서대로 신규 기능 개발, 버그 수정, 문서 산출물 작성 같은 작업을 진행할 때 사용한다.

작업 후 commit과 push는 동일하나 업로드 할 때 신규 브랜치로 하는 업로드는 처음일 경우
```
git push -u origin [카테고리/기능명]
```
의 형태로 진행해야한다.

## PR 요청
GitHub Web에서도 PR을 작성할 수 있지만 커맨드 명령어로도 가능하다.

```
gh pr create --title "PR 타이틀 작성" --body "PR 요청의 상세 내용 작성" --base main --head [브랜치 이름]
```
head는 생략이 가능하다.

## 합치기
마찬가지로 커맨드로 merge 기능을 수행할 수 있다.
```
gh pr merge --squash --delete-branch
git switch main
git pull --ff-only
```

실제로 합치는 기능을 하는 명령어는 가장 첫 줄로 main에 합치면서 업로드하는 브랜치(현재 작업 중이던 브랜치)를 삭제해준다.
main 브랜치로 돌아온 후 방금 업로드 한 작업을 가져오는 내용이다.

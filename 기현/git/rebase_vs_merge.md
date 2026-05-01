# Git rebase vs merge, 언제 뭘 써야 할까

브랜치로 작업하다 보면 결국
main(master)에 합쳐야 하는 순간이 온다.

이때 선택지는 두 가지다.

- merge
- rebase

둘 다 “합치는 것”이지만
--> 히스토리를 남기는 방식이 완전히 다르다

### merge — 흐름을 그대로 유지

```bash
git merge feature
```

**핵심**
- 브랜치 흐름 유지

- merge commit 생성

- 히스토리가 가지처럼 남음

### rebase — 히스토리를 정리

```bash
git rebase main
```
**핵심**

- 커밋을 다른 브랜치 위로 재배치

- 커밋이 새로 생성됨 (hash 변경)

- 히스토리가 일자로 정리됨

### 여기서 사고 많이 나니까 조심...
-> 이미 push한 브랜치에서 rebase

이유:

- 커밋이 바뀜

- 다른 사람 히스토리랑 달라짐

- pull / push 충돌 지옥

### 실무에서 이렇게 쓴다더라...

1. 혼자 작업 브랜치
```bash
git checkout feature
git rebase main
```

👉 충돌 미리 해결 + 히스토리 정리

2. 마지막에 merge
```Bash
git checkout main
git merge feature
```

한 줄 요약
> merge는 흐름을 남기고
> rebase는 흐름을 정리한다

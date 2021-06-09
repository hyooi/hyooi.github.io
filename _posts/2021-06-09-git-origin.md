---
layout: post
title:  "Git 원격 브랜치 사용"
---

### 원격 브랜치 갱신
```bash
git remote update
```


### 원격 브랜치 확인
```bash
git branch -a
```

### 원격 브랜치 체크아웃
```bash
git checkout -t origin/feature/344
```


### 원격 브랜치 참고
- detached HEAD상태로 소스 확인 및 수정 가능 
- 그러나 commit불가하며 다른 branch로 체크아웃 시 사라짐

```bash
git checkout origin/feature/344
```

### 원격 브랜치 삭제
```bash
git push origin --delete feature/344
```
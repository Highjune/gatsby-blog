---
title: 'git grep'
date: 2023-01-20 22:37:00
category: 'git'
draft: false
---

## 검색 (grep)
- 함수 정의나 함수가 호출되는 곳 검색해야 하는 경우 편함
- `-n` 옵션을 주면 찾은 문자열이 위치한 라인 번호도 같이 출력
```
git grep -n myfunction 
```
- `--count` 옵션은 결과 대신 어떤 파일에서 몇 개나 찾았는지만 알고 싶을 경우
```
git grep --count myfunction
```
- `-p` 옵션은 매칭되는 라인이 있는 함수나 메서드 찾고 싶을 때 
  - ex) date.c 라는 파일에서 myfunction 함수를 ~~ 에서 호출하고 있는 것을 확인 가능
```
git grep -p myfunction *.c
```
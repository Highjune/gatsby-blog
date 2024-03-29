---
title: '실행 계획'
date: 2023-07-11 22:48:00
category: 'database'
draft: false
---

> 실행 계획

## 실행 계획 확인 방법(mysql)
```
EXPLAIN SQL 구문
EXPLAIN ANALYZE SQL 구문
```

# 실행 계획에 나오는 중요한 3가지
## 1. 조작 대상 객체
- 실행계획 실행 후 on 뒤에 대상
- ex)
```
Table scan on Student
```

## 2. 객체에 대한 조작의 종류
- 가장 중요한 부분
- 어떤 식으로 접근했는지
	- 테이블 풀 스캔(= 시퀀셜 스캔)

## 3. 조작 대상이 되는 레코드 수
- rows 지표
- 카탈로그 매니저로부터 얻은 값. 즉 통계정보에서 파악한 숫자이므로, 실제 SQL 구문을 실행한 시점의 테이블 레코드 수와 차이가 있을 수 있다.
- 해당 테이블의 레코드르르 다 삭제하고, 실행 계획을 다시 검색하면 그대로 이전의 숫자(row 수)가 뜬다. 이는 옵티마이저가 통계라는 메타 데이터를 믿기 때문에 실제 테이블을 제대로 보지 않는다는 증거
	- 그런데, 직접 테스트해보니 지운 것이 반영된 수가 실제로 잘 나타났다. 이것은 아마 delete를 하면서 바로 카탈로그 매니저 정보도 업데이트하는 것이 아닐까 싶다.
---
title: '(mysql) 재귀쿼리로 계층구조 데이터 표현(id - varchar)'
date: 2021-04-16 17:30:00
category: 'database'
draft: false
---

# 재귀쿼리로 계층구조 데이터 표현하기(id(varchar)로)

- id에 varchar로 계층구조를 담아서 하지 않고 integer로 설계해서 표현하는 설계방법은 [여기로.]()
- mysql 기준으로 설명 (오라클은 CONNECT BY PRIOR, START WITH를 사용하여
  자신의 ID와 연결된 부모 ID를 찾아가 계층적으로 쿼리결과를 뽑을 수 있다.)
- 데이터를 계층구조(정렬순서포함)로 보여주고 싶을 때 사용
- ex) 메뉴

- Menu
  - 축구
    - 세리에A
    - K리그
    - 프리메라리그
  - 농구
    - NBA
    - KBL
  - 야구
    - MLB
    - KBO

## 테이블 설계

- (조건) id값은 VARCHAR로 설정
  - depth와 계층을 나타내기 위해서 필요

```
CREATE TABLE `recursion_test` (
	`id` VARCHAR(45) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`m_name` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`depth` INT(10) NULL DEFAULT NULL,
	`parent` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`m_order` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci'
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;
```

- 🧨단점)
  - 위처럼 id를 varchar로 하게 되면?
  - id를 생성할 때 이미 계층구조를 반영한 값들로 입력해야 한다.

## 데이터

- id값은 계층마다 규칙이 있어야 한다. (order의 기준이 되기 때문에)

- Menu(Root)
  - 축구(M0)
    - 세리에A(M01)
    - K리그(M01)
    - 프리메라리그(M01)
  - 농구(M1)
    - NBA(M10)
    - KBL(M11)
  - 야구(M2)
    - MLB(M20)
    - KBO(M21)

|  id  |    m_name    | depth | parent | m_oder |
| :--: | :----------: | :---: | :----: | :----: |
| ROOT |     MENU     |   0   | (NULL) |   0    |
|  M0  |     축구     |   1   |  ROOT  |   0    |
|  M1  |     농구     |   1   |  ROOT  |   1    |
|  M2  |     야구     |   1   |  ROOT  |   2    |
| M00  |   세리에A    |   2   |   M0   |   0    |
| M01  |   K-LEAGUE   |   2   |   M0   |   1    |
| M10  |     NBA      |   2   |   M1   |   0    |
| M11  |     KBL      |   2   |   M1   |   1    |
| M02  | 프리메라리그 |   2   |   M0   |   2    |
| M20  |     MLB      |   2   |   M2   |   0    |
| M21  |     KBO      |   2   |   M2   |   1    |

## 쿼리

- `WITH RECURSIVE` 문 사용

  - `WITH` 은 가상 테이블(임시) 생성 위해
  - `RECURSIVE` 은 재귀구현을 위해
    - 가상의 테이블이 그 자신(가상 테이블)을 참조하여 값을 결정하기 위해

- `WITH RECURSIVE` 규칙

  - RECURSIVE를 사용할 때는 서브 쿼리내에서 UNION (ALL) 이 사용되어야 한다.
  - 한 개 이상의 Non-Recursive 문장이 포함되어 있어야 한다. (첫번째 루프에서만 실행됨, `anchor` 역할임. 이것 미만의 계층구조로 연결되어 있는 것들 다 나옴)
  - 반복되는 Recursive문은 반드시 정지조건(where) 이 필요하다. (반복)

- 쿼리 실행1 (`메뉴(ROOT)`에 해당하는 계층 구조 다 확인)

```
SET @var = 'ROOT';

WITH RECURSIVE t3 (id, m_name, depth, parent, m_order) AS

(
	SELECT t1.id, t1.m_name, t1.depth, t1.parent, t1.m_order
	FROM practice.recursion_test t1
	WHERE t1.parent = @var

	UNION ALL

	SELECT t2.id, t2.m_name, t2.depth, t2.parent, t2.m_order
	FROM practice.recursion_test t2
	INNER JOIN t3 ON t2.parent = t3.id
)

SELECT * FROM t3
ORDER BY t3.id, t3.`m_order`;
```

결과는?

| id  |    m_name    | depth | parent | m_oder |
| :-: | :----------: | :---: | :----: | :----: |
| M0  |     축구     |   1   |  ROOT  |   0    |
| M00 |   세리에A    |   2   |   M0   |   0    |
| M01 |   K-LEAGUE   |   2   |   M0   |   1    |
| M02 | 프리메라리그 |   2   |   M0   |   2    |
| M1  |     농구     |   1   |  ROOT  |   1    |
| M10 |     NBA      |   2   |   M1   |   0    |
| M11 |     KBL      |   2   |   M1   |   1    |
| M2  |     야구     |   1   |  ROOT  |   2    |
| M20 |     MLB      |   2   |   M2   |   0    |
| M21 |     KBO      |   2   |   M2   |   1    |

- 쿼리 실행2 (`축구(M0)`에 해당하는 계층 구조 다 확인)

```
SET @var = 'M0';

WITH RECURSIVE t3 (id, m_name, depth, parent, m_order) AS

(
	SELECT t1.id, t1.m_name, t1.depth, t1.parent, t1.m_order
	FROM practice.recursion_test t1
	WHERE t1.parent = @var

	UNION ALL

	SELECT t2.id, t2.m_name, t2.depth, t2.parent, t2.m_order
	FROM practice.recursion_test t2
	INNER JOIN t3 ON t2.parent = t3.id
)

SELECT * FROM t3
ORDER BY t3.id, t3.`m_order`;
```

결과는?

| id  |    m_name    | depth | parent | m_oder |
| :-: | :----------: | :---: | :----: | :----: |
| M00 |   세리에A    |   2   |   M0   |   0    |
| M01 |   K-LEAGUE   |   2   |   M0   |   1    |
| M02 | 프리메라리그 |   2   |   M0   |   2    |

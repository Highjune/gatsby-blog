---
title: '(mysql + java) 재귀쿼리로 계층구조 데이터(JSON) 표현(id - Integer)'
date: 2021-04-21 17:30:00
category: 'database'
draft: false
---

# 재귀쿼리로 계층구조 데이터 표현하기(id(Integer)로)

- []()와 달리 id값을 Integer(Auto_increment)로 두게 되면 id값에 계층구조를 따로 담지 않아도 된다.
- 하지만 mysql 특성상 query 만으로 구현하기에는 까다롭기에 특정한 지점(편의상 노드라고 표현) 과 그의 자식(자식노드들)은 `재귀쿼리`로 들고와서 자바로 계층을 넣어서 `JSON`구조로 표현

- A
  - AA
    - AAA
    - AAB
    - AAC
  - AB
    - ABA
    - ABB
- B

## 테이블 설계

- (조건) id값은 Integer로 설정

```
CREATE TABLE `menu` (
	`id` INT(10) NOT NULL DEFAULT '0',
	`m_name` VARCHAR(45) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`depth` INT(10) NULL DEFAULT NULL,
	`parent_id` INT(10) NULL DEFAULT NULL,
	`m_order` INT(10) UNSIGNED NULL DEFAULT NULL,
	PRIMARY KEY (`id`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;
```

## 데이터

- ROOT의 parent_id는 NULL로 지정해도 된다.

| id  | m_name | depth | parent_id | m_oder |
| :-: | :----: | :---: | :-------: | :----: |
|  0  |  ROOT  |   0   |    -1     |   0    |
|  1  |   A    |   1   |     0     |   0    |
|  2  |   B    |   1   |     0     |   1    |
|  3  |   AA   |   2   |     1     |   0    |
|  4  |   AB   |   2   |     1     |   1    |
|  5  |  AAA   |   3   |     3     |   0    |
|  6  |  AAB   |   3   |     3     |   1    |
|  7  |  AAC   |   3   |     3     |   2    |
|  8  |  ABA   |   3   |     4     |   0    |
|  9  |  ABB   |   3   |     4     |   1    |

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

SET @num = 0;

SELECT id, m_name, depth, parent_id, m_order, url, multi_language_id, insert_time, update_time
FROM twadm.menu
WHERE 1=1
AND id = @num

UNION ALL

(
WITH RECURSIVE t3 (id, m_name, depth, parent_id, m_order, url, multi_language_id, insert_time, update_time) AS

(
	SELECT t1.id, t1.m_name, t1.depth, t1.parent_id, t1.m_order, t1.url, t1.multi_language_id, t1.insert_time, t1.update_time
	FROM twadm.menu t1
	WHERE t1.parent_id = @num

	UNION ALL

	SELECT t2.id, t2.m_name, t2.depth, t2.parent_id, t2.m_order, t2.url, t2.multi_language_id, t2.insert_time, t2.update_time
	FROM twadm.menu t2
	INNER JOIN t3 ON t2.parent_id = t3.id
)

SELECT * FROM t3
ORDER BY id, m_order
)


```

- 결과는?
  - 해당하는 id(ROOT=1) 과 그의 자식들 데이터는 다 나오지만, 정렬은 되어 있지 않다.

| id  | m_name | depth | parent_id | m_oder |
| :-: | :----: | :---: | :-------: | :----: |
|  0  |  ROOT  |   0   |    -1     |   0    |
|  1  |   A    |   1   |     0     |   0    |
|  2  |   B    |   1   |     0     |   1    |
|  3  |   AA   |   2   |     1     |   0    |
|  4  |   AB   |   2   |     1     |   1    |
|  5  |  AAA   |   3   |     3     |   0    |
|  6  |  AAB   |   3   |     3     |   1    |
|  7  |  AAC   |   3   |     3     |   2    |
|  8  |  ABA   |   3   |     4     |   0    |
|  9  |  ABB   |   3   |     4     |   1    |

- 쿼리 실행2 (`A(id=1)`에 해당하는 계층 구조 다 확인)

```

SET @num = 1;

SELECT id, m_name, depth, parent_id, m_order, url, multi_language_id, insert_time, update_time
FROM twadm.menu
WHERE 1=1
AND id = @num

UNION ALL

(
WITH RECURSIVE t3 (id, m_name, depth, parent_id, m_order, url, multi_language_id, insert_time, update_time) AS

(
	SELECT t1.id, t1.m_name, t1.depth, t1.parent_id, t1.m_order, t1.url, t1.multi_language_id, t1.insert_time, t1.update_time
	FROM twadm.menu t1
	WHERE t1.parent_id = @num

	UNION ALL

	SELECT t2.id, t2.m_name, t2.depth, t2.parent_id, t2.m_order, t2.url, t2.multi_language_id, t2.insert_time, t2.update_time
	FROM twadm.menu t2
	INNER JOIN t3 ON t2.parent_id = t3.id
)

SELECT * FROM t3
ORDER BY id, m_order
)

```

- 결과는?
  - (마찬가지0 해당하는 id(A=1) 과 그의 자식들 데이터는 다 나오지만, 정렬은 되어 있지 않다.

| id  | m_name | depth | parent_id | m_oder |
| :-: | :----: | :---: | :-------: | :----: |
|  0  |  ROOT  |   0   |    -1     |   0    |
|  1  |   A    |   1   |     0     |   0    |
|  2  |   B    |   1   |     0     |   1    |
|  3  |   AA   |   2   |     1     |   0    |
|  4  |   AB   |   2   |     1     |   1    |
|  5  |  AAA   |   3   |     3     |   0    |
|  6  |  AAB   |   3   |     3     |   1    |
|  7  |  AAC   |   3   |     3     |   2    |
|  8  |  ABA   |   3   |     4     |   0    |
|  9  |  ABB   |   3   |     4     |   1    |

## 자바에서의 작업

- controller와 reposiotry 는 생략
- 작업수행하는 service와 관련된 메서드만

### Menuservice.java

```
...중략...

@Autowired
    MenuDao menuDao;

public Map<String, Object> getMenuList(Map<String, Object> params) throws Exception {
		int upperNum = MapUtil.getNumber(params, "menu_id", 0).intValue();

		Map<String, Object> result = new HashMap<>();
		Map<String, Object> temp = new HashMap<>();

		List<HashMap<String, Object>> menuList = this.menuDao.selectMenuList(params);

		for (HashMap<String, Object> map : menuList) {
			try {

				int id = MapUtil.getNumber(map, "id", 0).intValue();
				int parent_id = MapUtil.getNumber(map, "parent_id", -1).intValue();

				if (temp.get("" + id) == null) {
					temp.put("" + id, map);
				}

				Map<String, Object> parent = MapUtil.getMap(temp, ""+parent_id);
				if (parent == null)
					continue;

				List<HashMap<String, Object>> list = MapUtil.getMapList(parent, "group");
				if (list == null) {
					list = new ArrayList<>();
					parent.put("group", list);
				}
				list.add(map);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}

		result.put("menulist", temp.get(""+upperNum));

		return result;
	}
```

### MapUtil.java

```
public static Number getNumber(Map<String, Object> map, String key, Number replace) {
		try {
			if (map == null || key == null || map.get(key) == null)
				return replace;
			Object value = map.get(key);
			if (value instanceof String) {
				if (((String) value).isEmpty())
					return replace;
				else
					return Double.parseDouble((value.toString()));
			} else {
				return (Number) value;
			}
		} catch (Exception e) {
		}
		return replace;
	}



public static HashMap<String, Object> getMap(Map<String, Object> map, String key) {
		try {
			if (map == null || key == null || map.get(key) == null)
				return null;
			return (HashMap<String, Object>) map.get(key);
		} catch (Exception e) {
		}
		return null;
	}



public static List<HashMap<String, Object>> getMapList(Map<String, Object> map, String key) {
		try {
			if (map == null || key == null || map.get(key) == null)
				return null;
			return (List<HashMap<String, Object>>) map.get(key);
		} catch (Exception e) {
		}
		return null;
	}
```

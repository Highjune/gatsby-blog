--- 
title: 'String 타입을 LocalDateTime으로 변환'
date: 2023-02-18 23:48:00
category: 'java'
draft: false
---


## 본론
```java
public static LocalDateTime stringToLocalDateTime(String stringDate) {
  String strDate = stringDate + " 00:00:00";
  return LocalDateTime.parse(strDate, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
}
```


## 배경
- 평소에 Api 작업을 할 때 `시작 ~ 끝` 조회를 하기 위해서 String 타입의 파라미터를 받는 경우가 많았다.
  - ex)
  ```java
  (..중략..)
  String startDate = requestDto.getStartDate(); // "2023-02-01"

  String endDate = requestDto.getEndDate(); // "2023-02-20"
  ```
  - 위처럼 받게 되었을 때 LocalDateTime 으로 변환하게 되면 DB 조회시 endDate에 +1를 더해서 아래처럼 계산하면 편하므로 LocalDateTime으로 변환한다.
  ```sql
  (Query 임)

  select *
    from A
    where A.createDate >= startDate
      and A.createDate < endDate
  ```
  - LocalDateTIme이 나오기 전에는 Date로 변환을 하기도 하고 했는데, LocalDateTime이 유용한 점이 많으므로 자주 사용하자.
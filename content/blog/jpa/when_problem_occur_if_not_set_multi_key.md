---
title: 'jpa 에서 복합키 설정을 하지 않은 경우 발생하는 문제점'
date: 2023-07-13 21:07:00
category: 'jpa'
draft: false
---

> DB에서는 복합키 설정이 되어 있지만 jpa 에서 복합키 설정을 하지 않은 경우

## 상황 설명
- DB에는 AgencyRestaurantRelation 테이블에 실제로 Agency relationIdx, agencyIdx, restaurantId 3가지가 같이 pk 로 잡혀있는 상황이라 치자.
- 그런데 아래코드에서 볼 수 있듯이 Entity에서는 1개만 pk(relationIdx) 를 설정한 상황이다.

```java

// Agency 는 여행사
// Restaurant은 식당

@Entity
public class AgencyRestaurantRelation {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long relationIdx;

    private Long agencyIdx;

    private String restaurantId;


// 생략
```

```java
public interface AgencyRestaurantRelationRepository extends JpaRepository<AgencyRestaurantRelation, Long> {
    // 생략

}
```

## 그러면 어떤 문제가 발생할까?
- agencyIdx, restaurantId 가 다르더라도 relationIdx 값만 같다면, key 값으로 영속성 컨텍스트 안에서 엔티티를 관리하는 jpa는 다 같은 값으로 처리한다.
- 데이터 예시
    - relationIdx, agencyIdx, restaurantId의 값이 각각
        - 데이터A : 1, 10, 20
        - 데이터B : 1, 17, 24
        - 데이터C : 1, 23, 32/
- 위처럼 있다고 가정했을 경우 AgencyRestaurantRelation 테이블의 데이터를 여러건 조회했을 때.
    - 제일 처음 조회가 된 것이 데이터A라면, 그 후에 데이터B, 데이터C는 key값인 1과 동일하다고 판단해 그대로 데이터 A로 간주해서 데이터A 로 덮어버린다. 그래서 아래와 같은 상황이 된다.
        - 데이터A : 1, 10, 20
        - 데이터A : 1, 10, 20
        - 데이터A : 1, 10, 20

## 그래서 복합키 설정을 꼭 해줘야 한다.
- 복합키 설정하는 방식은 다양하니 다른 방법으로 해도 된다.

```java
@Entity
@IdClass(AgencyRestaurantRelationKey.class)
public class AgencyRestaurantRelation {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long relationIdx;

    @Id
    private Long agencyIdx;

    @Id
    private String restaurantId;


// 생략
}

```

```java
@Embeddable
public class AgencyRestaurantRelationKey implements Serializable {
    private Long relationIdx;
    private Long officeIdx;
    private String storeId;
}
```

```java
public interface AgencyRestaurantRelationRepository extends JpaRepository<AgencyRestaurantRelation, AgencyRestaurantRelationKey> {

    
}
```
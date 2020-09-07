---
title: '컬렉션 프레임웍'
date: 2020-08-29 20:43:00
category: 'java'
draft: false
---
###### 자바의 정석을 공부하면서 정리한 것

## 컬렉션 프레임웍

### 컬렉션(collection)
- 여러 객체(데이터)를 모아 놓은 것을 의미

### 프레임웍(framework)
- 표준화, 정형화된 체계적인 프로그래밍 방식
- 자유도 낮은 대신 생산성 높음
- 방식이 비슷하기에 서로 협업하기에 편함

### 컬렉션 프레임웍 (collection framework)
- 컬렉션(다수의 객체)을 다루기 위한 표준화된 프로그래밍 방식
- 컬렉션을 쉽고 편리하게 다룰 수 있는(저장, 삭제, 검색, 정렬) 다양한 클래스를 제공
- java.util 패키지에 포함. jdk1.2부터 제공

### 컬렉션 클래스(collection class)
- 다수의 데이터를 저장할 수 있는 클래스
- ex) Vector, ArrayList, HashSet

### 컬렉션 프레임웍의 핵심 인터페이스(크게 3가지)
1. List
    - 순서 O, 중복 O
    - ex) 대기자 명단
    - 구현 클래스 : ArrayList, LinkedList, Stack, Vector 등

1. Set
    - 순서 X, 중복 X
    - ex) 양의 정수집합, 소수의 집합
    - 구현 클래스 : HashSet, TreeSet 등

1. Map
    - 키(key)와 값(value)의 쌍으로 이루어진 데이터의 집합
    - 순서 X, 키는 중복 X, 값은 중복 O
    - ex) 우편번호, 지역번호, id와 passwd
    - 구현 클래스 : HashMap, TreeMap, Hashtable, properties 등


- 위의 List와 Set의 공통부분만 추출해서 `Collection인터페이스`를 정의(Map은 성격이 약간 달라서)



### List 

![사진](./images/collectionframework1.png)
- ArrayList와 LinkedList가 핵심
- ArrayList 







참고 : 자바의 정석(기초편)

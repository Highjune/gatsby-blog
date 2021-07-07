---
title: 'Optional'
date: 2021-07-07 13:43:00
category: 'java'
draft: false
---

#### 자바의 정석을 공부하면서 정리한 것

# Lambda Expression(람다식)

- JDK1.8부터 추가됨
- 람다식의 도입으로 인해 자바는 객체지향언어인 동시에 함수형 언어가 됨

# 1. 람다식이란?

- 메서드를 하나의 식(expression)으로 표현한 것. 람다식은 함수를 간략하면서도 명확한 식으로 표현할 수 있게 해준다.
- 메서드를 람다식으로 표현하면 메서드의 이름과 반환값이 없어지므로, 람다식을 '익명 함수(anonymous function)'이라고도 한다.

```
int[] arr = new int[5];
Arrays.setAll(arr, (i) -> (int)(Math.random() * 5) + 1);
```

- 위에서 `(i) -> (int)(Math.random() * 5) + 1`이 바로 람다식이다. 이 람다식이 하는 일을 메서드로 표현하면 아래와 같다.

```
int method() {
    return (int)(Math.random()*5) + 1;
}
```

- 장점은?
  - 간결하고 이해하기 쉽다
  - 간단한 절차
    - 원래 모든 메서드는 클래스에 포함되어야 하므로 클래스도 새로 만들어야 하고, 객체도 생성해야만 비로소 이 메서드 호출할 수 있다.
    - 하지만 람다식은 이러한 모든 과정없이 오직 람다식 자체만으로도 이 메서드의 역할을 대신할 수 있다.
  - 메서드를 변수처럼 다룰 수 있다.
    - 람다식은 메서드의 매개변수로 전달되어지는 것이 가능하고 메서드의 결과로 반환될 수 있다.

# 2. 람다식 작성하기

- 람다식은 '익명 함수' 답게 메서드에서 이름과 반환타입을 제거하고 매개변수 선언부와 몸통 {} 사이에 `->`를 추가한다.

```
반환타입 메서드이름 (매개변수 선언) {
    문장들
}
```

- 위에서 반환타입과 메서드이름을 없애서 아래처럼 선언

```
(매개변수 선언) {
    문장들
}
```

- ex) 두 값 중에서 큰 값을 반환하는 메서드 max를 람다식으로 변환하면 다음과 같이 변화가 이루어진다.
  - 람다사용X
  ```
  int max (int a, int b) {
      return a > b ? a : b;
  }
  ```
  - 람다사용O
  ```
  (int a, int b) -> {
      return a > b ? a : b;
  }
  ```
- 반환값이 있는 메서드의 경우, return문 대신 '식(expression)으로 대신할 수 있다. 식의 연산결과가 자동적으로 반환값이 된다.이 때는 '문장(statement)'이 아닌 '식'이므로 끝에 ';' 을 붙이지 않는다.

  - return문 생략전

  ```
  (int a, int b) -> {return a > b ? a : b;}
  ```

  - return 문 생략 후, `;` 도 생략함

  ```
  (int a, int b) -> a > b ? a : b
  ```

- 람다식에 선언된 매개변수의 타입은 추론이 가능한 경우는 생략할 수 있는데, 대부분의 경우에는 생략가능하다. 람다식에 반환타입이 없는 이유도 항상 추론이 가능하기 때문이다.

  - 반환타입 생략 전

  ```
  (int a, int b) -> a > b ? a : b
  ```

  - 반환타입 생략 후

  ```
  (a, b) -> a > b ? : a : b
  ```

  - 주의) `(int a, b) -> a > b ? a : b`와 같이 두 매개변수 중 어느 하나의 타입만 생략하는 것은 허용되지 않는다.

- 선언된 매개변수가 하나뿐인 경우에는 괄호()를 생략할 수 있다. 단, 매개변수의 타입이 있으면 괄호()를 생략할 수 없다.
  - 괄호() 생략 전
  ```
  (a) -> a * a
  (int a) -> a * a
  ```
  - 괄호() 생략 후
  ```
  a -> a * a // OK
  int a -> a * a // Error
  ```
- 괄호{}안의 문장이 하나일 경우네는 괄호{}를 생략할 수 있다. 이 때 문장의 끝에 `;`를 붙이지 않아야 한다는 것에 주의

  - 괄호{} 생략 전

  ```
  (String name, int i) -> {
      System.out.println(name+"="+i);
  }
  ```

  - 괄호{} 생략 후, 그리고 끝에 `;` 생략함

  ```
  (String name, int i) ->
      System.out.println(name+"="+i)
  ```

  - 주의) 괄호{}안의 문장이 return문일 경우 괄호 {}를 생략할 수 없다.

  ```
  (int a, int b) -> { return a > b ? a : b;} // OK
  (int a, int b) ->  return a > b ? a : b // Error
  ```

- 메서드를 람다식으로 변환 연습1

  - 메서드

  ```
  int max (int a, int b) {
      return a > b ? a : b;
  }
  ```

  - 람다식으로 표현1

  ```
  (int a, int b) -> { return a > b ? a : b;}
  ```

  - 람다식으로 표현2

  ```
  (int a, int b) -> a > b ? a : b
  ```

  - 람다식으로 표현3

  ```
  (a, b) -> a > b ? a : b
  ```

- 메서드를 람다식으로 변환 연습2
  - 메서드
  ```
  void printVar (String name, int i) {
      System.out.println(name + "=" + i);
  }
  ```
  - 람다식으로 표현1
  ```
  (String name, int i) -> {System.out.println(name + "=" + i);}
  ```
  - 람다식으로 표현2
  ```
  (name, i) -> {System.out.println(name + "=" + i);}
  ```
  - 람다식으로 표현3
  ```
  (name, i) -> System.out.println(name + "=" + i)
  ```
- 메서드를 람다식으로 변환 연습3

  - 메서드

  ```
  int squre (int x) {
      return x * x;
  }
  ```

  - 람다식으로 표현1

  ```
  (int x) ->  x * x
  ```

  - 람다식으로 표현2

  ```
  (x) -> x * x
  ```

  - 람다식으로 표현3

  ```
  x -> x * x
  ```

- 메서드를 람다식으로 변환 연습4

  - 메서드

  ```
  int roll() {
      return (int) (Math.random() * 6);
  }
  ```

  - 람다식으로 표현1

  ```
  () -> {return (int) Math.random() * 6;}
  ```

  - 람다식으로 표현2

  ```
  () -> (int) Math.random() * 6
  ```

- 메서드를 람다식으로 변환 연습5

  - 메서드

  ```
  int sumArr (int[] arr) {
      int sum = 0;
      for (int i : arr)
        sum += i;
      return sum;
  }
  ```

  - 람다식으로 표현1

  ```
  (int[] arr) -> {
      int sum = 0;
      for (int i : arr)
        sum += i;
      return sum;
  }
  ```

# 3. 함수형 인터페이스(Functional Interface)

# 4. java.util.function 패키지

# 5. Function의 합성과 Predicate의 결합

# 6. 메서드 참조

#

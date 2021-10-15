---
title: 'Lambda'
date: 2021-10-25 14:00:00
category: 'java'
draft: false
---

#### 자바의 정석 기초 유투브 강의 듣고 정리한 것

# 람다(Lambda Expression)

## 람다란

- Java는 OOP 언어인데, JDK1.8 부터 함수형 언어의 기능을 탑재하게 되었다.
- Big Data를 처리하면서 함수형 언어가 새로 뜨기 시작
  - Haskell, Scala 등
- Python, JS 모두 OOP + 함수형 언어

### 함수(메서드)를 간단힌 식(Expression) 으로 표현하는 방법

```
int max(int a, int b) {
  return a > b ? a : b;
}
```

- 위를 아래로 표현(람다식으로)

```
(a, b) -> a > b ? a : b
```

### 익명 함수(이름이 없는 함수, anonymous function)

- 메서드를 람다식으로 수정시 반환 타입과 메서드 이름을 지운다. (그래서 익명함수라고 한다)
- `->` 화살표 생긴다

### 함수와 메서드의 차이

- 근본적으로 동일. 함수는 일반적 용어, 메서드는 객체지향개념 언어
- 함수는 클래스에 독립적, 메서드는 클래스에 종속적
- 자바는 모두 클래스 안에만 있으므로 모두 다 메서드. 클래스 밖에 존재할 수 없다.

### 람다식 작성하기

1. 메서드의 이름과 반환타입을 제거하고 `->`를 블록{} 앞에 추가한다.

```
int max(int a, int b) {
  return a > b ? a : b;
}
```

위에서 아래로

```
(int a, int b) -> {
  return a > b ? a : b;
}
```

2. 반환값이 있는 경우, 식이나 값만 적고 return문 생략 가능(끝에 `;` 안 붙임)

```
(int a, int b) -> {
  return a > b ? a : b
}

```

위에서 아래로

```
(int a, int b) -> a > b ? a : b
```

3. 매개변수의 타입이 추론 가능하면 생략가능(대부분의 경우 생략가능)

```
(int a, int b) -> a > b ? a : b
```

위에서 아래로

```
(a, b) -> a > b ? a : b
```

### 람다식 작성하기 - 주의사항

1. 매개변수가 하나인 경우, 괄호() 생략가능(타입이 없을 때만)

- `(a) -> a * a` 는 `a -> a * a` 로 가능하다
- `(int a) -> a * a` 는 `int a -> a * a`로 변경 불가능하다

2. 블록 안의 문장이 하나뿐일 때, 괄호 {} 생략가능 (끝에 `;` 안 붙임, 붙이면 안됨)

```
(int i) -> {
  System.out.println(i);
}
```

위에서 아래로 변경가능

```
(int i) -> System.out.println(i)
```

- 단 하나뿐인 문장이 return문이면 괄호 {} 생략불가(중요치 않음, 대부분 return을 생략하기 때문에)

```
(int a, int b) -> {return a > b ? a : b;} // OK
(int a, int b) -> return a > b ? a : b // 에러
```

### 람다식의 예(메서드 -> 람다식으로 변환)

1.

```
int max(int a, int b) {
  return a > b ? a : b;
}
```

위는 아래로 변경

```
(a, b) -> a > b : a : b
```

2.

```
int printVar(String name, int i) {
  System.out.println(name + "=" + i);
}
```

위는 아래로 변경

```
(name, i) -> System.out.println(name + "=" + i)
```

3.

```
int squre(int x) {
  return x * x;
}
```

위는 아래로 변경

```
x -> x * x
```

4.

```
int roll() {
  return (int)(Math.random() * 6);
}
```

위는 아래로 변경

```
() -> (int)(Math.random() * 6)
```

### 람다식은 익명 함수가 아니라 익명 객체이다

- 자바에서는 메서드만 따로 존재할 수 없다.

```
(a, b) -> a > b ? a : b
```

- 위는 사실 아래와 같다. 익명 객체임. 즉, 익명 클래스의 익명 객체임
  - 객체의 선언과 생성을 동시에.
  - 원래는 아래와 같이 써야 하는데 위처럼 람다식으로 간단히 표현한 것임
  - 그래서 람다식은 즉, 객체임
  ```
  new Object() {
    int max(int a, int b) {
      return a > b ? a : b;
    }
  }
  ```

### 람다식(익명 객체)을 다루기 위한 참조변수가 필요. 참조변수의 타입은?

- Object가 아니다.

```
Object obj = new Object() {
  int max(int a, int b) {
    return a > b ? a : b;
  }
};
```

```
타입 obj = (a, b) -> a > b ? a : b; // 어떤 타입일까?
int value = obj.max(3, 5); // 에러. Object클래스에 max()가 없음. Object리모콘으로 호출불가
```

- 자세한 건 이후에.

### 실습

```
public class Test {
    public static void main(String[] args) {
//        Object obb = (a, b) -> a > b ? a : b; // 에러. 함수형처럼 다뤄야 함. 자세한 것은 뒤에
        Object obj = new Object() { // 람다식은 객체이므로 이렇게 표현한 것. 하지만 object로 받으면 안된다.
            int max(int a, int b) {
                return a > b ? a : b;
            }
        };

        int value = obj.max(3, 5); // 에러. Object타입 리모콘에는 max가 없다.
    }
}
```

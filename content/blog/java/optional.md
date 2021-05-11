---
title: 'Optional'
date: 2021-05-11 20:43:00
category: 'java'
draft: false
---

#### 자바의 정석을 공부하면서 정리한 것

# Optional`<T>`

- T 타입 객체의 래퍼클래스 - Optional`<T>`

```
public final class Optional<T> {
  private final T value; // T타입의 참조변수
    ...
}
```

- 어떤 타입이든지 다 저장할 수 있다. 심지어 null도 가능.
- 왜 필요한가?
  1. null을 직접 다루는 것은 위험하기 때문에. nullPointerException 가능성
  2. null을 체크하기 위해서 if문을 사용하면 코드가 길어진다.
- 간접적으로 null을 다루는 것. null을 Optional 객체로 감싸는것. Optional은 객체이므로 주소가 존재한다. 그 객체(주소존재)가 가지고 있는 값이 null이라는 것. Optional이란 객체는 반드시 존재하기에 null일 없는 것
- 그림참조

```
String str = "abc";
Optional<String> optVal = Optional.of("abc");
```

![](./images/optional/optional.png)

- 참고1) String도 비슷

```
String str = null;
```

위와 같이 선언하지 않고 아래과 같이 빈 문자열로 선언한다.

```
String str = "";
```

위는 사실 아래와 같이 길이가 0인 character 배열이다.

```
String str = new char[0];
```

- 참고2) 배열 역시 마찬가지. 배열이 인스턴스 변수일 때 null로 초기화 하지 않고 빈 배열로 초기화 하는 것이 더 옳다.

```
int[] arr = null;
```

위보다 아래가 낫다.

```
int[] arr = new int[0];
```

## Optional`<T>` 객체를 생성하는 다양한 방법

```
String str = "abc";
Optional<String> optVal = Optional.of(str); // of는 static메서드.
Optional<String> optVal = Optional.of("abc");
Optional<String> optVal = Optional.of(null); // 에러임. NullPointerException 발생.
Optional<String> optVal = Optional.ofNullable(null) // 주로 이렇게 사용
```

## null대신 빈 Optional`<T>` 객체를 사용하자

```
Optional<String> optval = null; // 널로 초기화, 바람직하지 않음
Optional<String> optVal = Optional.<String>empty(); // 빈 객체로 초기화. 타입은 생략가능(아래처럼)
Optional<String> optVal = Optional.empty();

```

## Optional 객체의 값 가져오기 - get(), orElse(), orElseGet(), orElseThrow()

- 이중에서 orElsE(), orElseGet() 을 많이 사용한다.

```
Optional<String> optVal = Optional.of("abc");
String str1 = OptVal.get(); // optVal에 저장된 값을 반환, null이면 예외발생
String str2 = OptVal.orElse(""); // optVal에 저장된 값이 null일 때는, ""를 반환
String str3 = OptVal.orElseGet(String::new); // 람다식 사용가능 () -> new String()
String str4 = OptVal.orElseThrow(NullPointerException::new) // 널이면 예외발생, 예외종류 지정가능
```

## isPresent() - Optional객체의 값이 null이면 false, 아니면 true를 반환. null이 아닐때만 실행

```
if(Optional.ofnullable(str).isPresent()) { // if (str!=null)  와 같은 의미
  System.out.println(str);
}
```

아래와 같이 사용할 수 있다.

```
// ifPresent(Consumer) - 널이 아닐때만 작업 수행, 널이면 아무 일도 안 함
  Optional.ofNullable(str).ifPresent(Systme.out::println);
```

## 연습

- 초기화

```
//    int[] arr = null; // 이렇게 하면 NullPointerException 발생할 가능성 높음
      int[] arr = new int[0]; // 바람직O
      System.out.println("arr.length" + arr.length);

//    Optional<String> opt = null; // OK. 하지만 바람직X
      Optional<String> opt = Optional.empty(); // 바람직O
      System.out.println("opt=" + opt); // "opt=Optional.empty"
```

- get()

```
Optional<String> opt = Optional.empty();
System.out.println("opt=" + opt.get()); // 에러(NoSuchElementException). 이렇게 쓰면 안된다.
```

원래 잘 사용하진 않는 get()을 굳이 사용하기 위해서는?

```
String str = "";
      try {
          str = opt.get();
      } catch (Exception e) {
          str = ""; // 예외가 발생하면 빈 문자열("") 로 초기화
      }
```

- 번거로운 get() 대신에 orElsE() 사용

```
String str = "";
str = opt.orElse(""); // Optional 에 저장된 값이 null이면 빈 문자열 반환
str = opt.elElse("Empty 문자열");
```

- orElseGet() - 람다 사용가능

```
String str = "";
str = opt.orElseGet(() -> "Empty 문자열");
str = opt.orElseGet(String::new);
str = opt.orElseGet(() -> new String()); // 위를 이렇게 람다식으로 바꿀 수 있음
```

# OptinalInt, OptionalLong, OptionalDouble

## 기본형 값을 감싸는 래퍼클래스

```
public final class OptionalInt {
  ...
  private final boolean isPresent; // 값이 저장되어 있으면 true
  private final int value; // int타입의 변수
  ...
}

```

- Optional`<T>` 가 있는데도 이것을 굳이 사용하는 이유는? `성능` 때문에. 람다와 스트림이 모든 것을 감싸고 있고 객체로 다루고 있기 때문에 성능이 좀 떨어진다.

## Optional의 값 가져오기 - int getAsInt()

| Optional 클래스 | 값을 반환하는 메서드 |
| --------------- | -------------------- |
| Optional`<T>`   | T get()              |
| OptionalInt     | int getAsInt()       |
| OptionalLong    | long getAsLong()     |
| OptionalDouble  | double getAsDouble() |

## 빈 Optional 객체와의 비교

```
OptionalInt opt1 = OptionalInt.of(0); // Optional에 0을 저장함
OptionalInt opt2 = OptionalInt.empty(); // Optional에 0이 저장됨(기본형 초기화값)
```

위의 2개를 어떻게 구분?

```
OptionalInt opt1 = OptionalInt.of(0); // Optional에 0을 저장함
OptionalInt opt2 = OptionalInt.empty(); // Optional에 0이 저장됨(초기화)
System.out.println(opt1.isPresent()); // true
System.out.println(opt2.isPresent()); // false
System.out.println(opt1.equals()); // false
```

value는 결과적으로 0으로 같지만, isPresent() 값까지 같아야만 equals() 값이 true로 나오도록 Override되어있다.

## 실습

```
  Optional<String> optStr = Optional.of("abcde");
//  Optional<Integer> optInt = optStr.map(s->s.length());
  Optional<Integer> optInt = optStr.map(s->s.length()); // 위와 같음
  System.out.println("optStr = " + optStr.get()); // "optStr = abcde"
  System.out.println("optInt = " + optInt.get()); // "optInt = 5"

```

```
int result1 = Optional.of("123")
                    .filter(x->x.length() > 0)
                    .map(Integer::parseInt).orElse(-1);

int result2 = Optional.of("")
                    .filter(x->x.length() > 0)
                    .map(Integer::parseInt).orElse(-1); // 조건에 안 맞으니 map(Integer::parseInt)가 null이다. 그래서 -1반환

System.out.println("result1 = " + result1); // "result1 = 123"
System.out.println("result2 = " + result2); // "result1 = -1"
```

```
OptionalInt optInt1 = OptionalInt.of(0);
OptionalInt optInt2 = OptionalInt.empty();

System.out.println(optInt1.isPresent()); // true
System.out.println(optInt2.isPresent()); // false

System.out.println(optInt1.getAsInt()); // 0
//  System.out.println(optInt2.getAsInt()); // 에러(NoSuchElementException)
System.out.println("optInt1=" + optInt1);
System.out.println("optInt2=" + optInt2);
System.out.println("optInt1.equals(optInt2)?" + optInt1.equals(optInt2));
```

참고 : 자바의 정석 - 남궁성

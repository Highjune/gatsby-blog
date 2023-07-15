---
title: 'Lambda'
date: 2022-09-04 21:34:00
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

## 함수형 인터페이스

### 함수형 인터페이스 - 단 하나의 추상 메서드만 선언된 인터페이스

- 람다식을 다루기 위해서 사용됨
- 추상 메서드가 2개 이상이면 에러남(@FunctionalInterface 애너테이션을 붙인다면)

```
@FunctionalInterface // 애너테이션은 안 붙여도 되는데 붙이면, 붙이면 함수형 인터페이스를 올바르게 작성했는지를 체크해준다. 그래서 붙여주는 게 좋다
interface MyFunction {
  public abstract int max(int a, int b);
}
```

- 위를 아래처럼 구현

```
MyFunction f = new MyFunction() { // 익명클래스{}의 선언과 객체 생성을 동시에
                  public int max(int a, int b) {
                      return a > b ? a : b;
                  }
              };
```

- 그러면 아래처럼 사용가능
  - Object로 선언하지 않았기 때문에 가능
  ```
  int value = f.max(3, 5); // OK. MyFunction에 max() 가 있음
  ```

### 함수형 인터페이스 타입의 참조변수로 람다식을 참조할 수 있음

- 단, 함수형 인터페이스의 메서드와 람다식의 `매개변수 개수, 타입`와 `반환타입`이 일치해야 함. 즉 람다(익명 객체이자 메서드)는 이름을 없앴기 때문에 상기 조건을 일치시켜야만 함수형 인터페이스의 메서드와 연결해서 사용할 수 있다(무슨 메서드인지는 알아야 하므로). 추상 메서드를 통해서 람다식을 호출하는 것

```
MyFunction f = (a, b) -> a > b ? a : b;
int value = f.max(3, 5);  // 실제로는 람다식(익명 함수)이 호출됨
```

- 람다식이 익명객체, 즉 객체이므로 참조변수(f)가 필요하다.(람다식을 다루려면)
  - 그 참조변수의 타입이 되는 것이 바로 함수형 인터페이스 타입

### 실습

```
public class Test {
    public static void main(String[] args) {
//        MyFunction f = new MyFunction() { // 람다식. 익명객체
//
//            @Override
//            public int max(int a, int b) { // public붙인 이유는 오버라이딩시 접근제어자를 좁게 못 바꾸기 때문에
//                return a > b ? a : b;
//            }
//        };

        // 람다식(익명 객체) 을 다루기 위한 참조변수의 타입은 함수형 인터페이스로 한다.
        MyFunction f = (a, b) -> a > b ? a : b; // 람다식. 익명객체

        int value = f.max(3, 5);
        System.out.println("value = " + value);
    }
}

@FunctionalInterface
interface MyFunction {
    int max(int a, int b); // public abstract int max(int a, int b); 와 동일.
}

```

## 함수형 인터페이스의 예

### 익명 객체를 람다식으로 대체

- 원래는 아래처럼 구현했어야 했다

```
List<String> list = Arrays.asList("abc", "aaA", "bbb", "ddd", "aaa");

Collections.sort(list, new Comparator<String>() {
                        public int compare(String s1, String s2) {
                          return s2.compareTo(s1);
                        }
                      });
```

```
// Comparator<T> 참고

// @FunctionalInterface 생략되어 있음
interface Comparator<T> {
  int compare(T o1, T o2);
}
```

- 아래로 변환 가능 (간단히)
  - Comparator가 함수형 인터페이스라 가능

```
List<String> list = Arrays.asList("abc", "aaA", "bbb", "ddd", "aaa");
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

## 함수형 인터페이스 타입의 매개변수, 반환타입

### 함수형 인터페이스 타입의 매개변수

- 파라미터가 함수형 인터페이스(MyFunction) 이란 것은 즉, 람다식을 받겠다는 뜻

```
@FunctionalInterface
interface MyFunction {
  void myMethod();
}
```

```
void aMethod(MyFunction f) {
  f.myMethod(); // MyFunction에 정의된 메서드 호출. 즉 람다식 호출
}
```

- 그래서 아래와 같이 사용가능하다

```
MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);
```

- 위를 아래처럼 변경가능. 람다식을 직접 넣은 것

```
aMethod(() -> System.out.println("myMethod()");
```

### 함수형 인터페이스 타입의 반환타입

- 반환타입이 함수형 인터페이스라는 것은 즉, 람다식을 반환하겠다는 것

```
MyFunction myMethod() {
  MyFunction f = () -> {}'
  return f;
}
```

- 위는 아래처럼 변경가능

```
MyFunction myMethod() {
  return () -> {};
}
```

### 실습 (람다식 주고받는)

```
@FunctionalInterface
interface MyFunction {
    void run(); // public abstarct void run();
}

public class Test {
    static void execute(MyFunction f) { // 매개변수의 타입이 MyFunction(함수형 인터페이스) 인 메서드
        f.run();
    }

    static MyFunction getMyFunction() { // 반환 타입이 MyFunction인 메서드. 즉 람다식을 반환
//        MyFunction f = () -> System.out.println("f3.run()");
//        return f;
        // 위 2줄을 아래 1줄로 변환 가능
        return () -> System.out.println("f3.run()");
    }

    public static void main(String[] args) {
        // 1. 람다식으로 MyFunction의 run()을 구현
        MyFunction f1 = () -> System.out.println("f1.run()"); // void run()과 파라미터가 없다는 것, 반환타입도 없다는 것 다 일치.

        // 2. 익명클래스로 MyFunction의 run()을 구현
        MyFunction f2 = new MyFunction() { //
            @Override // @Override 안 붙여도 되긴 함
            public void run() { // public을 반드시 붙여야 함
                System.out.println("f2.run()");
            }
        };

        // 3. MyFunction 인터페이스의 run()을 구현한 것을 반환(람다로) 하는 것을 받기
        MyFunction f3 = getMyFunction();

        f1.run();   // f1.run();
        f2.run();   // f2.run();
        f3.run();   // f3.run();

        execute(f1);    // f1.run(); , execute(() -> System.out.println("f1.run()")); 와 동일하다
        execute(() -> System.out.println("run()")); // run()
    }
}


```

## java.util.function 패키지

### 자주 사용되는 다양한 함수형 인터페이스를 제공

![image](https://user-images.githubusercontent.com/57219160/138211525-05aa2d59-f78c-4899-9aa6-6c11feeca350.png)

- 사실 우리가 사용하는 함수의 종류가 많지 않으므로 그것들을 미리 만들어 놓은 것. 매번 만드는 것보다 자바가 만든 것을 우리가 사용하는 것이 더 편리. 그러면 표준화가 된다는 장점
- Supplier는 공급자, Consumer 는 소비자, Function은 함수, Predicate는 조건식

- Predicate`<T>` 사용예제

  ```
  Predicate<String> isEmptyStr = s -> s.length() == 0;
  String s = "";

  if (isEmptyStr.test(s)) // if(s.length() == 0)
    System.out.println("This is an empty String.");
  ```

  - 조건식이 boolean타입이어야 한다.
    - s.length() == 0;
  - `String s`가 아니라 `s`인 이유는 앞에서 Predicate`<String>` 이미 String인 것을 알고 있으므로
  - `if (isEmptyStr.test(s))` 에서 test는 `s -> s.length() == 0;` 식을 말한 것이다. 이 식에 test라는 이름이 붙은 것임. 이 람다식을 사용하려면 test라는 이름을 사용해야만 하는 것

- 예시(퀴즈)
  - 4개의 람다식을 다루기 위한 함수형 인터페이스(java.util.function 패키지)를 적으시오
  1. `Supplier<Integer>` f = () -> (int)(Math.Random()\*100) + 1; // Integer인 이유는 반환타입이 Integer이므로. int -> Integer로 오토박싱되어서 반환(지네릭스는 primitive 타입 못 씀)
  2. `Consumer<i>` f = i -> System.out.print(i + ", "); // 반환값 없음. 그냥 콘솔에 출력뿐. i 타입이므로 Integer
  3. `Predicate<Integer>` f = i -> i % 2 == 0; // 조건식이므로 Predicate. 원래는 Function`<T, R>`처럼 Predicate`<Integer, Boolean>`이라고 써야 하지만 반환타입이 항상 Boolean 이기 때문에 Boolean은 쓰지 않는다.
  4. `Function<Integer, Integer>` f = i -> i / 10 \* 10; // Integer를 넣으면 Integer가 반환되어서 `<Integer, Integer>`

### 매개변수가 2개인 함수형 인터페이스

![image](https://user-images.githubusercontent.com/57219160/138213857-4948277d-f83c-44c6-80f2-6516eaf248d9.png)

- 매개변수가 2개면 앞에 `Bi`가 붙는다.
- BiSupplier는 2개를 반환해야 하므로, 불가능하기에 없다.

- BiConsumer`<T, U>`는 T, U를 받기만 할 뿐 반환은 없고, BiPredicate`<T, U>`는 T, U를 받아서 Boolean을 반환하고, BiFunction`<T, U, R>`은 T, U를 받아서 R을 반환한다.

- 매개변수 3개부터는 없다. 만약 3개 이상이 필요하다면 아래처럼 직접 만들어야 한다.
  - T, U, V 3개를 받는 인터페이스를 아래처럼 만들어야 한다.
  ```
  @FunctionalInterface
  interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
  }
  ```
- 매개변수가 1~2개일 경우에는 직접 만들어서 쓰지 말고 기존의 제공하는 것 사용하기.

### 매개변수의 타입과 반환타입이 일치하는 함수형 인터페이스

![image](https://user-images.githubusercontent.com/57219160/138214559-c8c4f70a-27b7-4218-bc71-ee07e8c4d66a.png)

- UnaryOperator`<T>` 는 단항연산자, BinaryOperator`<T>`는 이항연산자는 뜻
  - int -> int 처럼 단항연산자, int + int -> int 처럼 이항연산자
- UnaryOperator`<T>` 소스코드

```
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

  static <T> UnaryOperator<T> identity() {
    return t -> t;  // 항등함수
  }
}
```

- Function`<T, R>` 소스코드

```
@FunctionalInterface
pubic interface Function<T, R> {

  R apply(T t);
  ....
}
```

### 실습

```
public class Test {
    public static void main(String[] args) {
        Supplier<Integer> s = () -> (int)(Math.random()*100) + 1; // 1~100난수
        Consumer<Integer> c = i -> System.out.print(i + ", ");
        Predicate<Integer> p = i -> i % 2==0; // 짝수인지 검사
        Function<Integer, Integer> f = i -> i / 10 * 10; // i의 일의 자리를 없앤다.

        List<Integer> list = new ArrayList<>();
        makeRandomList(s, list); // s를 이용해 list를 1~100 랜덤값으로 채운다.
        System.out.println(list);

        printEvenNum(p, c, list); // 짝수를 출력
        List<Integer> newList = doSomething(f, list);
        System.out.println(newList);
    }

    static <T> List<T> doSomething(Function<T, T> f, List<T> list) {
        List<T> newList = new ArrayList<T>(list.size());

        for (T i : list) {
            newList.add(f.apply(i)); // 일의 자리를 없애서 새로운 list에 저장
        }

        return newList;
    }

    static <T> void printEvenNum(Predicate<T> p, Consumer<T> c, List<T> list) {
        System.out.print("[");
        for (T i : list) {
            if(p.test(i)) { // Predicate<Integer> p = i -> i % 2==0; // 짝수인지 검사
                c.accept(i); //    Consumer<Integer> c = i -> System.out.print(i + ", ");
            }
        }
        System.out.println("]");
    }

    static <T> void makeRandomList(Supplier<T> s, List<T> list) {
        for (int i = 0 ; i < 10 ; i++) {
            list.add(s.get()); // Supplier로부터 1~100의 난수를 받아서 list에 추가
        }
    }
}
```

## Predicate의 결합

### and(), or(), negate()로 두 Predicate를 하나로 결합(default메서드)

```
Predciate<Integer> p = i -> i < 100;
Predciate<Integer> q = i -> i < 200;
Predciate<Integer> r = i -> i%2 == 0;
```

- 위의 조건들을 아래와 같이 결합할 수 있다.

```
Predciate<Integer> notP = p.negate(); // i >= 100
Predicate<Integer> all = notP.and(q).or(r); // 100 <= i && i < 200 || i % 2 == 0
Predicate<Integer> all2 = notP.and(q.or(r)); // 100 <= i && (i < 200 || i % 2 == 0)
```

- Predicate 가 가지고 있는 추상 메서드인 test로 테스트해볼 수 있다.

```
System.out.println(all.test(2)); // true
System.out.println(all2.test(2)); // false
```

### 등가비교를 위한 Predicate의 작성에는 isEqual() 를 사용(static 메서드)

- 참고) 인터페이스에는 default 메서드, static 메서드, 추상 메서드 3가지를 가질 수 있다. 추상메서드가 메인
- 아래처럼 사용가능

```
Predicate<String> p = Predicate.isEqual(str1); // isEquals() 은 static메서드
Boolean result = p.test(str2); // str1과 str2가 같은지 비교한 결과를 반환
```

- 위는 아래 1줄로 축약 가능

```
boolean reuslt = Predicate.isEqual(str1).test(str2);
```

### 실습
- andThen(), compose() 함수 포함

```
public class Test {
    public static void main(String[] args) {
        // 위에서 개념 설명은 없었지만 문제를 통해서 설명
        // 함수 2개를 합치는 것. f함수와 g를 합치려면 f의 출력(Integer)과 g의 입력(Integer)가 동일해야 한다.

        Function<String, Integer> f = (s) -> Integer.parseInt(s, 16); // String s를 16진수로 변경
        Function<Integer, String> g = (i) -> Integer.toBinaryString(i); // 2진수 문자열로 변경

        // h는 f와 g를 합친 새로운 함수 h를 만든 것임 (String -> Integer -> String)
        Function<String, String> h = f.andThen(g); // f 적용 후 g를 적용 하는 것이 andThen (함수를 2개 붙인 것)
        // h2는 g와 f를 합친 새로운 함수 h2를 만든 것임 (Integer -> String -> Integer)
        Function<Integer, Integer> h2 = f.compose(g); // g 적용 후 f를 적용 (즉, g.andThen(f)와 같고 andThen을 쓰는 것이 더 가독성 좋음)

        System.out.println(h.apply("FF")); // "FF" -> 255 -> "11111111"
        System.out.println(h2.apply(2)); // 2 -> "10" -> 16

        Function<String, String> f2 = x -> x; // 항등 함수(identity function)
        System.out.println(f2.apply("AAA")); // AAA가 그대로 출력됨

        Predicate<Integer> p = i -> i < 100;
        Predicate<Integer> q = i -> i < 200;
        Predicate<Integer> r = i -> i % 2 == 0;
        Predicate<Integer> notP = p.negate(); // i >= 100

        Predicate<Integer> all = notP.and(q.or(r)); // (i>=100) && (i<200 || i%2==0)
        System.out.println(all.test(150)); // true

        String str1 = "abc";
        String str2 = "abc";

        // str1과 str2가 같은지 비교한 결과를 반환
        Predicate<String> p2 = Predicate.isEqual(str1);
        boolean result = p2.test(str2);  // 위 2줄을 boolean result = Predicate.isEqual(str1).test(str2); 와 같다. 이것은 boolean result = str1.equals(str2); 와 역시 같다.
        System.out.println(result); // True
        // 위에서 String str1 = new String("abc");, String str2 = new String("abc"); 해도 결과는 똑같다. 그 말은 등가비교(==) 를 하는 것이 아니라 equals로 비교한다는 것을 의미

    }
}

```

## 컬렉션 프레임워크와 함수형 인터페이스

### 함수형 인터페이스를 사용하는 컬렉션 프레임웍의 메서드(와일드 카드 생략)

![image](https://user-images.githubusercontent.com/57219160/138637466-49347b8d-24f6-47bd-b437-24f0cd28cdd4.png)

```
list.forEach(i -> System.out.print(i + ",")); // list의 모든 요소를 출력
list.removeIf(x -> x%2 == 0 || x % 3 == 0); // 2 또는 3의 배수를 제거
list.replaceAll(i -> i * 10); // 모든 요소에 10을 곱한다.

// map의 모든 요소를 {k, v} 의 형식으로 출력
map.forEach((k, v) -> System.out.println("{" + k + "," + v + "} , "));
```

- 그동안 컬렉션 프레임워크가 그대로 사용하기에는 불편한 점들이 많았는데(코드들도 길고), JDK 1.8부터 람다식을 이용해서 사용할 수 있는 메서드들이 제공되어서 덕분에 코드가 간단해진다.

### 실습

```
public class Test {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        for (int i = 0 ; i < 10 ; i++) {
            list.add(i);
        }

        // 1. list의 모든 요소를 출력. (물론 System.out.println(list); 만 해도 잘 나오긴 함.)
        // 기존에는 아래처럼 코드가 길었음.
/*
        Iterator<Integer> it = list.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
         }
*/
        // 람다식 이용할 수 있는 메서드 덕분에 간결해짐
        list.forEach(i -> System.out.println(i + ",")); // forEach는 consumer들어가니 받기만 하는 것

        // 2. list에서 2 또는 3의 배수를 제거한다.
        list.removeIf(x -> x%2 == 0 || x%3 == 0);
        System.out.println(list);

        // 3. list의 각 요소에 10을 곱한다.
        list.replaceAll(i -> i*10);
        System.out.println(list);

        // 4. map의 모든 요소를 {k, v} 의 형식으로 출력한다.
        Map<String, String> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");

        // 출력
        // 기존코드는 매우 길다
/*
        Iterator it = map.entrySet().iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
*/
        // 람다식 이용할 수 있는 메서드 덕분에 간결해짐
        map.forEach((k, v) -> System.out.println("{" + k + "," + v + "},"));
        System.out.println();
    }
}
```

## 메서드 참조(method reference)

### 하나의 메서드만 호출하는 람다식은 '메서드 참조'로 간단히 할 수 있다.

- `클래스이름::메서드이름`
- 람다식을 더 간단하게.
- 아래 3가지 중 특정 객체 인스턴스메서드 참조는 잘 안 씀

![image](https://user-images.githubusercontent.com/57219160/138809128-a6064e6b-eefe-46bc-949f-311bc50bc17a.png)

### static메서드 참조

- A

```
Integer method(String s) { // 그저 Integer.parseInt(String s)만 호출
  return Integer.parseInt(s);
}
```

- 즉, 아래 메서드는.

```
int result = obj.method("123");
```

- 아래와 동일하다는 것.(어차피 하나의 메서드만 호출하므로)

```
int result = Integer.parseInt("123");
```

- 따라서 A는 아래처럼 차례로 변할 수 있다. (람다식 <--> 메서드 참조, 양방향 변환을 연습해야 한다)
  - 우선 람다식으로 변환
  ```
  Function<String, Integer> f = (String s) -> Integer.parseInt(s); // 람다식
  ```
  - 메서드 참조로 다시 변환(람다식에서 어차피 입력값이 String인 것 아니까 `(String s)` 와 parseInt(s)의 `(s)`는 생략해도 됨, 특히 parseInt(s)는 컴파일러가 애초에 무엇을 받는지 미리 다 알고 있으므로.
    ), 그래서 다 지우면 Integer.paserInt; 가 남는데 .를 ::로 문법상 약속
  ```
  Function<String, Integer> f = Integer::parseInt; // 메서드 참조
  ```

### 실습

```
public class Test {
    public static void main(String[] args) {
//        Function<String, Integer> f = (String s) -> Integer.parseInt(s); // 람다식
        Function<String, Integer> f = Integer::parseInt; // 위의 람다식을 메서드 참조로 변환
        System.out.println(f.apply("100"));
    }
}
```

## 생성자의 메서드 참조

### 생성자와 메서드 참조

- 생성자에 매개변수가 없는 경우

  - 람다식

    - 제공만 하는 Supplier니까 입력이 없이 출력(여기서는 클래스)만 있음.
    - 입력이 없으니까 람다식에서 ()

    ```
    Supplier<Myclass> s = () -> new MyClass();
    ```

  - 메서드 참조(위의 람다식을 변환)

    - 입력 자체가 없으므로 () 다 삭제. 그리고 메서드 이름은 new니까 new

    ```
    Supplier<MyClass> s = MyClass::new;
    ```

- 생성자에 매개변수가 있는 경우
  - 람다식
    - 입력값 Integer타입 i를 주고 출력이 MyClass 클래스
    ```
    Function<Integer, MyClass> s = (i) -> new Myclass(i);
    ```
  - 메서드 참조(위의 람다식을 변환)
    - 어차피 입력값 Integer인 것을 아니까 (i)삭제, new Myclass에서도 입력값인 i를 받을 것을 아니까 (Function에서 준다고 했으니까) 삭제
    - 매개변수가 2개면 BiFunction`<T, U, R>` 사용하면 됨. T, U가 입력값, R이 반환값
    ```
    Function<Integer, MyClass> s = MyClass::new;
    ```

### 배열과 메서드 참조 (자주 사용)

- 람다식
  - 배열길이(x, 입력값) 가 필요하므로 Function 써야 한다.
  ```
  Function<Integer, int[]> f = x -> new int[x]; // 람다식
  ```
- 메서드 참조(위의 람다식을 변환)
  - 어차피 입력값 Integer를 아니까
  ```
  Function<Integer, int[]> f2 = int[]::new; // 메서드 참조
  ```

### 실습

```
public class Test {
    public static void main(String[] args) {
        // Supplier은 입력X, 출력O
        Supplier<MyClass> s1 = () -> new MyClass(); // 람다식(파라미터 X)
        Supplier<MyClass> s2 = MyClass::new; // 메서드 참조(파라미터 X)
        System.out.println(s2.get()); // MyClass mc = s2.get();로 객체 얻어서 System.out.println(mc); 한 것임

        // Function은 입력O, 출력O
        Function<Integer, MyClass> f1 = (i) -> new MyClass(i); // 람다식(파라미터 O)
        Function<Integer, MyClass> f2 = MyClass::new; // 메서드 참조(파라미터 O)
        MyClass mc = f1.apply(100);
        System.out.println(f1.apply(100).iv);  //100, System.out.println(mc.iv); 와 같음

        // 배열. 배열을 반환(출력)하려면 배열의 길이(입력)를 줘야 하니까 Function
        Function<Integer, int[]> r1 = (k) -> new int[k]; // 람다
        Function<Integer, int[]> r2 = int[]::new;   // 메서드 참조
        System.out.println(r2.apply(100).length); // 100
    }
}

class MyClass {
    int iv;

    MyClass(int iv) {
        this.iv = iv;
    }

    MyClass() {}
}
```

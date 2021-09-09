---
title: 'Lambda(자바의 정석 3판 책 정리)'
date: 2021-07-29 14:42:00
category: 'java'
draft: false
---

#### 자바의 정석(3판, 책)을 공부하면서 정리한 것

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

- 람다식은 익명 클래스의 객체와 동일하다.

  - 왜냐하면 사실 자바에서 모든 메서드는 클래스 내에 포함되어야 하므로.

- 그래서 아래의 람다식은 사실 익명 클래스의 객체로 표현가능하다.
  - 람다로
  ```
  (int a, inb b) -> a > b ? a : b;
  ```
  - 익명 클래스의 객체로
  ```
  new Object() {
      int max (int a, int b) {
          return a > b ? a : b;
      }
  }
  ```
- 위의 코드에서 메서드 이름 max는 임의로 붙인 것일 뿐 의미는 없다. 어쨌든 람다식으로 정의된 익명 객체의 메서드를 어떻게 호출할 것인가? 이미 알고 있는 것처럼 참조변수가 있어야 객체의 메서드를 호출할 수 있으니까 일단 이 익명 객체의 주소를 f라는 참조변수에 저장해보자

```
타입 f = (int a, int b) -> a > b ? a : b; // 참조변수의 타입을 뭘로 해야 할까?
```

- 그러면, 참조변수 f의 타입은 어떤 것이어야 하나? 참조형이므로 클래스 또는 인터페이스가 가능함. 그리고 람다식과 동등한 메서드가 정의되어 있는 것이어야 한다. 그래야만 참조변수로 익명 객체(람다식)의 메서드를 호출할 수 있기 때문이다.
- 예를 들어, 아래와 같이 max()라는 메서드가 정의된 MyFunction인터페이스가 정의되어 있다고 가정하자.

```
interface MyFunction {
    public abstract int max(int a, int b);
}
```

그러면 이 인터페이스를 구현한 익명 클래스의 객체는 다음과 같이 생성 가능

```
MyFunction f = new Function() {
    public int max(int a, int b) {
        return a > b ? a : b;
    };
}
int big = f.max(3, 5); // 익명 객체의 클래스를 호출
```

그런데 위의 MyFunction인터페이스에 정의된 메서드 max()는 람다식 `(int a, int b) -> a > b ? a : b`과 메서드의 선언부가 일치한다. 그래서 위 코드의 익명 객체를 람다식으로 아래와 같이 대체할 수 있다.

```
MyFunction f = (int a, int b) -> a > b ? a : b; // 익명 객체를 람다식으로 대체
ing big = f.max(5, 3); // 익명 객체의 메서드를 호출
```

- (중요) 이처럼 MyFunction인터페이스를 구현한 익명 객체를 람다식으로 대체가 가능한 이유는, 람다식도 실제로는 익명 객체이고, MyFunction인터페이스를 구현한 익명 객체의 메서드 max()와 람다식의 매개변수의 타입과 개수 그리고 반환값이 일치하기 때문이다. 하나의 메서드가 선언된 인터페이스를 정의해서 람다식을 다루는 것은 기존의 자바의 규칙들을 어기지 않으면서도 자연스러움.
- 그래서 인터페이스를 통해 람다식을 다루기로 결정된 것이며, 람다식을 다루기 위한 인터페이스를 `함수형 인터페이스(functional interface)`라고 부르기로 한 것.

```
@FunctionalInterface
interface MyFunction { // 함수형 인터페이스 MyFunction을 정의
    public abstract int max(int a, int b);
}
```

- 제약) 단, 함수형 인터페이스에는 오직 하나의 추상 메서드만 정의되어 있어야 한다. 그래야 람다식과 인터페이스의 메서드가 1:1로 연결될 수 있기 때문. 반면에 static메서드와 default메서드의 개수에는 제약이 없다.

  - 참고) `@FunctionalInterface` 를 붙이면, 컴파일러가 함수형 인터페이스를 올바르게 정의하였는지 확인해주므로, 꼭 붙이자

- 기존에는 아래와 같이 인터페이스의 메서드 하나를 구현하는데도 복잡하게 했어야 했는데,

```
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaA");

Collections.sort(list, new Comparator<String>(){
    public int compare(String s1, String s2) {
        return s2.compareTo(s1);
    }
});
```

- 이제는 람다식으로 아래와 같이 간단히 처리할 수 있게 되었다

```
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

## 함수형 인터페이스 타입의 매개변수와 반환타입

- 함수형 인터페이스 MyFunction이 아래와 같이 정의되어 있을 때,

```
@FunctionalInterface
interface MyFunction {
  void myMethod(); // 추상메서드
}
```

- 메서드의 매개변수가 MyFunction타입이면, 이 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야 한다는 뜻이다.

```
void aMethod(MyFunction f) {
  f.myMethod();
}
  ...
MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);
```

- 또는 참조변수 없이 아래와 같이 직접 람다식을 매개변수로 지정하는 것도 가능

```
aMethod(() -> System.out.println("myMethod()")); // 람다식을 매개변수로 지정
```

- 그리고 메서드의 반환타입이 함수형 인터페이스타입이라면, 이 함수형 인터페이스의 추상메서드와 동등한 람다식을 가리키는 참조변수를 반환하거나 람다식을 직접 반환할 수 있다.

```
MyFunction myMethod() {
  MyFunction f = () -> {};
  return f;     // 이 줄과 윗 줄을 한 줄로 줄이면, return () -> {};
}
```

- 이렇게 람다식을 참조변수로 다룰 수 있다는 것은 메서드를 통해 람다식을 주고받을 수 있다는 것을 의미한다. 즉, 변수처럼 메서드를 주고받는 것이 가능해진 것이다. 사실상 메서드가 아니라 객체를 주고받는 것이므로 근본적으로 달라진 것은 아무것도 없지만, 람다식 덕분에 예전보다 코드가 더 간결하고 이해하기 쉬워졌다.

- 예제 14-1)

```
@FunctionalInterface
interface MyFunction {
    void run();
}

class LambdaEx1 {
    static void execute(MyFunction f) { // 매개변수의 타입이 MyFunction인 메서드
        f.run();
    }

    static MyFunction getMyFunction() { // 반환 타입이 MyFunction인 메서드
        MyFunction f = () -> System.out.println("f3.run()");
        return f;
    }

    public static void main(String[] args) {
        // 람다식으로 MyFunction의 run() 을 구현
        MyFunction f1 = () -> System.out.println("f1.run()");

        MyFunction f2 = new MyFunction() { // 익명클래스로 run()을 구현
            public void run() { // public을 반드시 붙여야 함
                System.out.println("f2.run()");
            }
        };

        MyFunction f3 = getMyFunction();
        f1.run();
        f2.run();
        f3.run();

        execute(f1);
        execute(() -> System.out.println("run()"));
    }

}

/////////////////////////////////
(출력)
f1.run()
f2.run()
f3.run()
f1.run()
run()
```

## 람다식의 타입과 형변환

- 함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다. 람다식은 익명 객체이고 익명 객체는 타입이 없다. 정확히는 타입은 있지만 컴파일러가 임의로 이름을 정하기 때문에 알 수 없는 것이다. 그래서 대입 연산자의 양변의 타입을 일치시키기 위해 아래와 같이 형변환이 필요하다.

- 참고) MyFunction은 `interface MyFunction {void method();}` 와 같이 정의되었다고 가정하였다.

```
MyFunction f = (MyFunction) (() -> {}); // 양변의 타입이 다르므로 형변환이 필요
```

- 람다식은 MyFunction인터페이스를 직접 구현하지 않았지만, 이 인터페이스를 구현한 클래스의 객체와 완전히 동일하기 때문에 위와 같은 형변환을 허용한다. 그리고 이 형변환은 생략가능하다.
- 람다식은 이름이 없을 뿐 분명히 객체인데도, 아래와 같이 Object타입으로 형변환 할 수 없다.람다식은 오직 함수형 인터페이스로만 형변환이 가능하다.

```
Object object = (Object) (() -> {}); // 에러. 함수형 인터페이스로만 형변환 가능
```

- 굳이 Object타입으로 형변환하려면, 먼저 함수형 인터페이스로 변환해야 한다.

```
Object obj = (Object)(MyFunction) (() -> {});
String str = ((Object)(MyFunction) (() -> {})).toString();
```

- 예제 14-2)

```

@FunctionalInterface
interface MyFunction {
    void myMethod(); // public abstract void myMethod();
}

public class LambdaEx2 {
    public static void main(String[] args) {
        MyFunction f = () -> {}; // MyFunction f = (MyFunction)(() -> {});
        Object obj = (MyFunction) (() -> {}); // Object 타입으로 형변환이 생략됨
        String str = ((Object)(MyFunction)(() -> {})).toString();

        System.out.println(f); // LambdaEx2$$Lambda$1/1324119927@3d075dc0
        System.out.println(obj); // LambdaEx2$$Lambda$2/1078694789@214c265e
        System.out.println(str); // LambdaEx2$$Lambda$3/1831932724@682a0b20

//        System.out.println(() -> {}); // 에러. 람다식은 Object타입으로 형변환 안됨
        System.out.println((MyFunction)(() -> {})); // LambdaEx2$$Lambda$4/1149319664@7cca494b
//        System.out.println((MyFunction)(() -> {}).toString()); // 에러
        System.out.println(((Object)(MyFunction)(() -> {})).toString()); // LambdaEx2$$Lambda$5/2074407503@3b9a45b3
    }

}
```

- 예제 14-2) 에서 볼 수 있듯이 실행결과를 보면, 컴파일러가 람다식의 타입을 어떤 형식으로 만들어 내는지 알 수 있다.
  - 일반적인 익명 객체일 경우 객체의 타입 : `외부클래스$번호`
  - 람다식의 타입 : `외부클래스$$Lambda$번호

## 외부 변수를 참조하는 람다식

- 람다식도 익명 객체, 즉 익명 클래스의 인스턴스이므로 람다식에서 외부에 선언된 변수에 접근하는 규칙은 익명 클래스에서 배운 것과 동일하다.

- 예제 14-3)

```
import com.sun.org.apache.xerces.internal.dom.PSVIAttrNSImpl;

@FunctionalInterface
interface MyFunction {
    void myMethod();
}

class Outer {
    int val = 10;  // Outer.this.val

    class Inner {
        int val = 20; // this.val

        void method(int i) { // void method(final int i) { 와 동일
            int val = 30; // final int val = 30;
//            i = 10; // 에러. 상수의 값을 변경할 수 없음.

            MyFunction f = () -> {
                System.out.println("                 i : " + i);  // 100
                System.out.println("               val : " + val); // 30
                System.out.println("          this.val : " + ++this.val); // 21
                System.out.println("    Outer.this.val : " + ++Outer.this.val); // 11
            };

            f.myMethod();
        }
    } // Inner클래스 끝
} // Outer 클래스 끝

public class LambdaEx3 {
    public static void main(String[] args) {
        Outer outer = new Outer();
        Outer.Inner inner = outer.new Inner();
        inner.method(100);
    }
}
```

- 예제 14-3) 에서는, 람다식 내에서 참조하는 지역변수는 final이 붙지 않았어도 상수로 간주된다. 람다식 내에서 지역변수 i와 val을 참조하고 있으므로 람다식 내에서나 다른 어느 곳에서도 값을 변경하는 일은 허용되지 않는다. 반면에 Inner클래스와 Outer클래스의 인스턴스 변수인 this.val와 Outer.this.val은 상수로 간주되지 않으므로 값을 변경해도 된다.

  - 외부 지역변수와 같은 이름의 람다식 매개변수는 허용되지 않는다. (아래에서 에러2)

  ```
  void method(int i) {
    int val = 30; // final int val = 30;
    i = 10; // 에러1. 상수의 값을 변경할 수 없음.

    MyFunction f = (i) -> { // 에러2. 외부 지역변수와 이름이 중복됨.
        System.out.println("                 i : " + i);
        System.out.println("               val : " + val);
        System.out.println("          this.val : " + ++this.val);
        System.out.println("    Outer.this.val : " + ++Outer.this.val);
    }
  }
  ```

# 4. java.util.function 패키지

- 대부분의 메서드는 타입이 비슷하다. 매개변수가 없거나 한 개 또는 두 개, 반환 값은 없거나 한 개. 게다가 지네릭 메서드로 정의하면 매개변수나 반환 타입이 달라도 문제가 되지 않는다. 그래서 java.util.function 패키지에 일반적으로 자주 쓰이는 형식의 메서드를 함수형 인터페이스로 미리 정의해 놓은 것. `매번 새로운 함수형 인터페이스를 정의하지 말고, 가능하면 이 패키지의 인터페이스를 활용하는 것이 좋다.`
- 그래야 함수형 인터페이스에 정의된 메서드 이름도 통일되고, 재사용성이나 유지보수 측면에서도 좋다. 자주 쓰이는 가장 기본적인 함수형 인터페이스는 다음과 같다. (참고로 타입 문자 `T`는 `Type`을, `R`은 `Return Type`을 의미한다)

| 함수형 인터페이스  | 메서드                                  | 설명                                                             |
| ------------------ | --------------------------------------- | ---------------------------------------------------------------- |
| java.lang.Runnable | `void run()`                            | 매개변수도 없고, 반환값도 없음                                   |
| Supplier`<T>`      | `void accept(T t)` --T-->               | 매개변수도 없고, 반환값만 있음                                   |
| Consumer`<T>`      | --T--> `void accept(T t)`               | Supplier와 반대로 매개변수만 있고, 반환값이 없음                 |
| Function`<T,R>`    | --T--> `R apply(T t)` --R-->            | 일반적인 함수, 하나의 매개변수를 받아서 결과를 반환              |
| Predicate`<T>`     | --T--> `boolean test(T t)` --boolean--> | 조건식을 표현하는데 사용됨. 매개변수는 하나, 반환 타입은 boolean |

- 매개변수와 반환값의 유무에 따라 4개의 함수형 인터페이스가 정의되어 있고, Function의 변형으로 Predicate가 있는데, 반환값이 boolean이라는 것만 제외하면 Function과 동일하다. Predicate는 조건식을 함수로 표현하는데 사용된다.

## 조건식의 표현에 사용되는 Predicate

- Predicate는 Function의 변형으로, 반환타입이 boolean이라는 것만 다르다. Predicate는 조건식을 람다식을 표현하는데 사용된다.
  - 참고) 수학에서 결과로 true 또는 false를 반환하는 함수를 '프레디케이트(predicate)'라고 한다.

```
Predicate<String> isEmptyStr = s -> s.length() == 0;
String s = "";

if (isEmptyStr.test(s)) // if(s.length()) == 0
  System.out.println("This is an empty String.");
```

## 매개변수가 두 개인 함수형 인터페이스

- 매개변수의 개수가 2개인 함수형 인터페이스는 이름 앞에 접두사 'Bi'가 붙는다.
  - 참고) 매개변수의 타입으로 보통 `T`를 사용하므로, 알파벳에서 `T` 다음 문자인 `U`, `V`, `W`를 매개변수의 타입으로 사용하는 것일 뿐 별다른 의미는 없다.

| 함수형 인터페이스   | 메서드                                          | 설명                                                        |
| ------------------- | ----------------------------------------------- | ----------------------------------------------------------- |
| BiConsumer`<T,U>`   | --T, U--> `void accept(T t, U u)`               | 두 개의 매개변수만 있고, 반환값이 없음                      |
| BiPredicate`<T,U>`  | --T, U--> `boolean test(T t, U u)` --boolean--> | 조건식을 표현하는데 사용됨. 매개변수는 둘, 반환값은 boolean |
| BiFunction`<T,U,R>` | --T, U--> 'R apply(T t, U u)' --R-->            | 두 개의 매개변수를 받아서 하나의 결과를 반환                |

- 참고) Supplier는 매개변수는 없고 반환값만 존재하는데, 메서드는 두 개의 값을 반환할 수 없으므로 BiSupplier가 없는 것이다.
- 두 개 이상의 매개변수를 갖는 함수형 인터페이스가 필요하다면 직접 만들어서 써야한다. 만일 3개의 매개변수를 갖는 함수형 인터페이스를 선언한다면 다음과 같을 것이다.

```
@FunctionalInterface
interface TriFunction<T, U, V, R> {
  R apply(T t, U u, V v);
}
```

## UnaryOperator와 BinaryOperator

- Function의 또 다른 변형으로 UnaryOperator와 BinaryOperator가 있는데, 매개변수의 타입과 반환타입의 타입이 모두 일치한다는 점만 제외하고는 Function과 같다.
- 참고) UnaryOperator와 BinaryOperator의 조상은 각각 Function와 BiFunction이다.

| 함수형 인터페이스   | 메서드                               | 설명                                                                |
| ------------------- | ------------------------------------ | ------------------------------------------------------------------- |
| UnaryOperator`<T>`  | --T--> `T apply(T t) --T-->          | Function의 자손, Function과 달리 매개변수와 결과의 타입이 같다.     |
| BinaryOperator`<T>` | --T, T--> `T apply(T t, T t)` --t--> | BiFunction의 자손, BiFunction과 달리 매개변수와 결과의 타입이 같다. |

## 컬렉션 프레임웍과 함수형 인터페이스

- 컬렉션 프레임웍의 인터페이스에 다수의 디폴트 메서드가 추가되었는데, 그 중의 일부는 함수형 인터페이스를 사용한다. 다음은 그 메서드들의 목록이다.(컬렉션 프레임웍의 함수형 인터페이스를 사용하는 메서드들)
  - 참고) 단순화하기 위해 와일드 카드는 생략하였다.

| 인터페이스 | 메서드                                             | 설명                             |
| ---------- | -------------------------------------------------- | -------------------------------- |
| Collection | boolean removeIf(Predicate`<E>` filter)            | 조건에 맞는 요소를 삭제          |
| List       | void replaceAll(UnaryOperator`<E>` operator)       | 모든 요소를 변환하여 대체        |
| Iterable   | void forEach(Consumer`<T>` action)                 | 모든 요소에 작업 action을 수행   |
| Map        | V compute(K key, BiFunction`<K, V, V>` f)          | 지정된 키의 값에 작업 f를 수행   |
| Map        | V computeIfAbsent(K key, Function`<K, V>` f)       | 키가 없으면, 작업 f 수행 후 추가 |
| Map        | V computeIfPresent(K key, BiFunction`<K, V, V>` f) | 지정된 키가 있을 때, 작업 f 수행 |
| Map        | V merge(K key, V value, BiFunction`<V, V, V>` f)   | 모든 요소에 병합작업 f를 수행    |
| Map        | void forEach(BiConsumer`<K, V>` action)            | 모든 요소에 작업 action을 수행   |
| Map        | void replaceAll(BiFunction`<K, V, V>` f)           | 모든 요소에 치환작업 f를 수행    |

- 이름만 봐도 어떤 일을 하는 메서드인지 충분히 알 수 있을 것이다. Map인터페이스에 있는 'Compute'로 시작하는 메서드들은 맵의 value를 변환하는 일을 하고 merge()는 Map을 병합하는 일을 한다. 이 메서드들을 어떤 식으로 사용하는지는 다음의 예제를 보자.

- 예제 14-4)

```
public class LambdaEx4 {
    public static void main(String[] args) {
        ArrayList<Integer> list = new ArrayList<>();
        for (int i = 0 ; i < 10 ; i++) {
            list.add(i);
        }

        // list 모든 요소 출력
        list.forEach(i -> System.out.print(i + ",")); // 0,1,2,3,4,5,6,7,8,9,
        System.out.println();

        // list에서 2 또는 3의 배수를 제거한다.
        list.removeIf(x -> x % 2==0 || x % 3 ==0); // [1, 5, 7]
        System.out.println(list);

        list.replaceAll(i -> i * 10); // list의 각 요소에 10을 곱한다.
        System.out.println(list); // [10, 50, 70]

        Map<String, Object> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        map.put("3", "3");
        map.put("4", "4");

        // map의 모든 요소를 {k, v} 의 형식으로 출력한다.
        map.forEach((k, v) -> System.out.print("{" + k + " ," + v + "},")); // {1 ,1},{2 ,2},{3 ,3},{4 ,4},
        System.out.println();
    }
}
```

- 메서드의 기본적인 사용법만 보여주는 간단한 예제. 예제의 람다식을 변형해서 다양하게 테스트해보기

- 다음은 앞서 설명한 함수형 인터페이스들을 사용하는 예제.
- 예제 14-5)

```
import com.sun.scenario.effect.impl.sw.sse.SSEBlend_SRC_OUTPeer;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class LambdaEx5 {

    public static void main(String[] args) {
        Supplier<Integer> s = () -> (int)(Math.random() * 100) + 1;
        Consumer<Integer> c = i -> System.out.print(i + ", ");
        Predicate<Integer> p = i -> i%2 == 0;
        Function<Integer, Integer> f = i -> i/10*10; // i의 1의 자리를 없앤다.

        List<Integer> list = new ArrayList<>();
        makeRandomList(s, list);
        System.out.println(list); //  랜덤숫자 뽑음 [79, 78, 84, 9, 39, 4, 15, 21, 99, 13]
        printEvenNum(p, c, list); //  뽑은 것 중에서 짝수만 출력 [78, 84, 4, ]
        List<Integer> newList = doSomething(f, list);
        System.out.println(newList); // 랜덤숫자들 중 1의 자리 없앰[70, 70, 80, 0, 30, 0, 10, 20, 90, 10]
    }

    static <T> List<T> doSomething(Function<T, T> f, List<T> list) {
        List<T> newList = new ArrayList<T>(list.size());

        for (T i : list) {
            newList.add(f.apply(i));
        }
        return newList;
    }

    static <T> void printEvenNum(Predicate<T> p, Consumer<T> c, List<T> list) {
        System.out.print("[");
        for (T i : list) {
            if (p.test(i))
                c.accept(i);
        }
        System.out.println("]");
    }

    static <T> void makeRandomList(Supplier<T> s, List<T> list) {
        for (int i = 0 ; i < 10 ; i++) {
            list.add(s.get());
        }
    }

}
```

## 기본형을 사용하는 함수형 인터페이스

- 지금까지 소개한 함수형 인터페이스는 매개변수와 반환값의 타입이 모두 지네릭 타입이었는데, 기본형 타입의 값을 처리할 때도 래퍼(wrapper)클래스를 사용해왔다. 그러나 기본형 대신 래퍼클래스를 사용하는 것은 당연히 비효율적이다. 그래서 보다 효율적으로 처리할 수 있도록 기본형을 사용하는 함수형 인터페이스들이 제공된다.

| 인터페이스            | 메서드                                         | 설명                                                 |
| --------------------- | ---------------------------------------------- | ---------------------------------------------------- |
| Double TolIntFunction | --double--> `int applyAsInt(double d)`--int--> | `A`To`B`Function은 입력이 A타입 출력이 B타입         |
| ToIntFunction`<T>`    | --T--> `int applyAsInt(T value)` --int-->      | To`B`Function은 출력이 B타입이다. 입력은 지네릭 타입 |
| IntFunction`<T>`      | --int--> `R apply(T t, U u)` --R-->            | `A`Function은 입력이 A타입이고 출력은 지네릭 타입    |
| ObjIntConsumer`<T>`   | --T, int--> `void accept(T t, U u)`            | Obj`A`Function은 입력이 T, A타입이고 출력은 없다.    |

- 예제 14-6) 예제 14-5를 기본형을 사용하는 함수형 인터페이스로 변경한 것

```
import java.util.Arrays;
import java.util.function.IntConsumer;
import java.util.function.IntPredicate;
import java.util.function.IntSupplier;
import java.util.function.IntUnaryOperator;

public class LambdaEx6 {
    public static void main(String[] args) {
        IntSupplier s = () -> (int)(Math.random()*100) + 1;
        IntConsumer c = i -> System.out.print(i + ", ");
        IntPredicate p = i -> i % 2 == 0;
        IntUnaryOperator op = i -> i/10*10; // i의 일의 자리를 없앤다.

        int[] arr = new int[10];

        makeRandomList(s, arr);
        System.out.println(Arrays.toString(arr)); // [27, 29, 54, 82, 71, 91, 62, 63, 51, 8]
        printEvenNum(p, c, arr); // [54, 82, 62, 8, ]
        int[] newArr = doSomething(op, arr);
        System.out.println(Arrays.toString(newArr)); // [20, 20, 50, 80, 70, 90, 60, 60, 50, 0]
    }


    static void makeRandomList (IntSupplier s, int[] arr) {
        for (int i = 0 ; i < arr.length ; i++) {
            arr[i] = s.getAsInt(); // get() 이 아니라 getAsInt() 임에 주의
        }
    }

    static void printEvenNum (IntPredicate p, IntConsumer c, int[] arr) {
        System.out.print("[");
        for (int i : arr) {
            if (p.test(i))
                c.accept(i);
        }
        System.out.println("]");
    }

    static int[] doSomething(IntUnaryOperator op, int[] arr) {
        int[] newArr = new int[arr.length];

        for (int i = 0 ; i < newArr.length ; i++) {
            newArr[i] = op.applyAsInt(arr[i]); // apply()가 아님에 주의
        }

        return newArr;
    }
}
```

- 위 예제에서 만일 아래와 같이 IntUnaryOperator 대신 Function 을 사용하면 에러가 발생한다.

```
Function f = (a) -> 2*a; // 에러. a의 타입을 알 수 없으므로 연산불가
```

- 매개변수 a와 반환 값의 타입을 추정할 수 없기 때문이다. 그래서 아래와 같이 타입을 지정해 주어야 한다.

```
// OK, 매개변수 타이봐 반환타입이 Integer
Function<Integer, Integer> f = (a) -> 2*a;
```

- 또는 아래와 같이 Function 대신 IntFunction을 사용할 수도 있지만, IntUnaryOperator 가 Function이나 IntFunction보다 오토박싱&언박싱의 횟수가 줄어들어 더 성능이 좋다.

```
// OK, 매개변수 타입은 int, 반환타입은 Integer
IntFunction<Integer> f = (a) -> 2 * a;
```

- IntFunction, ToIntFunction, IntToLongFunction은 있어도 IntToIntFunction은 없는데, 그 이유는 IntUnaryOperator가 그 역할을 하기 때문이다. 매개변수의 타입과 반환타입이 일치할 때는 앞서 배운 것처럼 Function 대신 UnaryOperator를 사용하자.

# 5. Function의 합성과 Predicate의 결합

- 앞서 소개한 java.util.function 패키지의 함수형 인터페이스는 추상메서드 외에도 디폴트 메서드와 static메서드가 정의되어 있다. 우리는 Function과 Predicate에 정의된 메서드에 대해서만 살펴볼 것인데, 그 이유는 다른 함수형 인터페이스의 메서드도 유사하기 때문이다. 이 두 함수형 인터페이스에 대한 설명만으로도 충분하니 응용이 가능할 것이다.
- 참고) 원래 Function인터페이스는 반드시 두 개의 타입을 지정해 줘야하기 때문에, 두 타입이 같아도 Function`<T>` 라고 쓸 수 없다. Function`<T, T>` 라고 써야 한다.

- Function
  - default `<V>` Function<T, V> `andThen`(Function<? super R, ? extends V> after)
  - default `<V>` Function<V, R> `compose`(Function<? super V, ? extends T> before)
  - static `<T>` Function<T, T> `identify`()
- Predicate
  - default Predicat`<T>` `and`(Predicate<? super T> other)
  - default Predicat`<T>` `or`(Predicate<? super T> other)
  - default Predicat`<T>` `negate()`
  - static `<T>` Predicate`<T>` isEqual(Object targetRef)

## Function의 합성

- 수학에서 두 함수를 합성해서 하나의 새로운 함수를 만들어낼 수 있는 것처럼, 두 람다식을 합성해서 새로운 람다식을 만들 수 있다. 이미 알고 있는 것처럼, 두 함수의 합성은 어느 함수를 먼저 적용하느냐에 따라 달라진다. 함수 f, g가 있을 때, f.andThen(g)는 함수 f를 먼저 적용하고, 그 다음에 함수 g를 적용한다. 그리고 f.compose(g)는 반대로 g를 먼저 적용하고 f를 적용한다.
- (자바의 정석 p809 그림14-1에 상세히 나와있음)
  - andThen()과 compose() 의 비교
    - default `<V>` Function<T, V> andThen(Function<? super R, ? extends V> after)
    - dfault `<V>` Function<V, R> compose(Function<? super V, ? extends T> before)
- 예를 들어, 문자열을 숫자로 변환하는 함수 f와 숫자를 2진 무자열로 변환하는 함수 g를 andThen()으로 합성하여 새로운 함수 h를 만들어낼 수 있다.
  - Function<`String`, Integer> f = (s) -> Integer.parseInt(s, 16);
  - Function<Integer, `String`> g = (i) -> Integer.toBinaryString(i);
  - Function<`String`, `String`> h = f.andThen(g);
- 함수 h의 지네릭 타입이 `<String, String>` 이다. 즉, String을 입력받아서 String을 결과로 반환한다. 예를 들어 함수 h에 문자열 "FF"를 입력하면, 결과로 "11111111"을 얻는다.
  - System.out.println(h.apply("EE")); // "FF" -> 255 -> "11111111"
- 이번엔 compose()를 이용해서 두 함수를 반대의 순서로 합성해보자.
  - Function<Integer, String> g = (i) -> Integer.toBinaryString(i);
  - Function<String, Integer> f = (s) -> Integer.parseInt(s, 16);
  - Function<Integer, Integer> h = f.compose(g);
- 이전과 달리 함수 h의 지네릭 타입이 `<Integer, Integer>` 이다. 함수 h에 숫자 2를 입력하면, 결과로 16을 얻는다.
  - 참고) 함수 f는 "10"을 16진수로 인식하기 때문에 16을 결과로 얻는다.
  - Systme.out.println(h.apply(2)); // 2 -> "10" -> 16
- 그리고 identify()는 함수를 적용하기 이전과 이후가 동일한 '항등 함수'가 필요할 때 사용한다. 이 함수를 람다식으로 표현하면 'x -> x' 이다. 아래의 두 문장은 동등하다.
  - 참고) 항등 함수는 함수에 x를 대입하면 결과가 x인 함수를 말한다. f(x) = x
  ```
    - Function<String, String> f = x -> x;
    // Function<String, String> f = Function.identify(); // 위의 문장과 동일
  System.out.println(f.apply("AAA")); // AAA가 그대로 출력됨
  ```
  - 항등 함수는 잘 사용되지 않는 편이며, 나중에 배울 map()으로 변환작업할 때, 변환없이 그대로 처리하고자 할 때 사용된다.

## Predicate의 결합

- 여러 조건식을 논리 연산자인 &&(and), ||(or), !(not)으로 연결해서 하나의 식을 구성할 수 있는 것처럼, 여러 Predicate를 and(), or(), negate()로 연결해서 하나의 새로운 Predicate로 결합할 수 있다.

```
Predicate<Integer> p = i -> i < 100;
Predicate<Integer> q = i -> i < 200;
Predicate<Integer> r = i -> i % 2 == 0;
Predicate<Integer> notP = p.negate(); // i >= 100

// 100 <= i && (i < 200 || i%2 ==0)
Predicate<Integer> all = notP.and(q.or(t));
System.out.println(all.test(150)); // true
```

- 이처럼 and(), or(), negate()로 여러 조건식을 하나로 합칠 수 있다. 물론 아래와 같이 람다식을 직접 넣어도 된다.

```
Predicate<Integer> all = notP.and(i -> i < 200).or(i -> i % 2 == 0);
```

- 주의) Predicate의 끝에 negate()를 붙이면 조건식 전체가 부정이 된다.
- 그리고 static 메서드인 isEqual()은 두 대상을 비교하는 Predicate를 만들 때 사용한다. 먼저 isEqual()의 매개변수로 비교대상을 하나 지정하고, 또 다른 비교대상은 test()의 매개변수로 지정한다.

```
Predciate<String> p = Predicate.isEqual(str1);
boolean result = p.test(str2); // str1과 str2과 같은지 비교하여 결과를 반환
```

- 위의 두 문장을 합치면 아래와 같다. 오히려 아래의 문장이 이해하기 더 쉬울 것이다.

```
// str1과 str2가 같은지 비교
boolean result = Predicate.isEqual(str1).test(str2);
```

- 예제 14-7)

```
import java.util.function.Function;
import java.util.function.Predicate;

public class LambdaEx7 {
    public static void main(String[] args) {
        Function<String, Integer> f = (s) -> Integer.parseInt(s, 16);
        Function<Integer, String> g = (i) -> Integer.toBinaryString(i);

        Function<String, String> h = f.andThen(g);
        Function<Integer, Integer> h2 = f.compose(g);

        System.out.println(h.apply("FF")); // "FF" -> 255 -> "11111111"
        System.out.println(h2.apply(2)); // 2 -> "10" -> 16

        Function<String, String> f2 = x -> x; // 항등함수(identity function)
        System.out.println(f2.apply("AAA")); // AAA가 그대로 출력됨

        Predicate<Integer> p = i -> i < 100;
        Predicate<Integer> q = i -> i < 200;
        Predicate<Integer> r = i -> i % 2 == 0;
        Predicate<Integer> notP = p.negate(); // i >= 100

        Predicate<Integer> all = notP.and(q.or(r));
        System.out.println(all.test(150)); // true

        String str1 = "abc";
        String str2 = "abc";

        // str1과 str2가 같은지 비교한 결과를 반환
        Predicate<String> p2 = Predicate.isEqual(str1);
        boolean result = p2.test(str2);
        System.out.println(result); // true
    }
}

```

# 6. 메서드 참조

- 람다식으로 메서드를 이처럼 간결하게 표현할 수 있는 것만 해도 장점. 그러나 람다식을 더욱 간결하게 표현할 수 있는 방법이 잇다. 항상 그런 것은 아니고, 람다식이 하나의 메서드만 호출하는 경우에는 `메서드 참조(method reference)`라는 방법으로 람다식을 간략히 할 수 있다. 예를 들어 문자열을 정수로 변환하는 람다식은 아래와 같이 작성할 수 있다.

```
Function<String, Integer> f = (String s) -> Integer.parseInt(s);
```

- 보통은 위처럼 람다식을 작성하는데, 이 람다식을 메서드로 표현하면 아래와 같다.
  - 참고) 람다식은 엄밀히 말하자면 익명클래스의 객체지만 간단히 메서드만 적었다.
  ```
  Integer wrapper(String s) { // 이 메서드의 이름은 의미없다.
    return Integer.parseInt(s);
  }
  ```
- 위의 wrapper 메서드는 별로 하는 일이 없다.그저 값을 받아서 Integer.parseInt()에게 넘겨주는 일만 할 뿐이다. 차라리 이 거추장스러운 메서드를 벗겨내고 Integer.parseInt()를 직접 호출하는 것이 더 낫지 않을까?

  - 즉 아래를,

  ```
  Function<String, Integer> f = (String s) -> Integer.parseInt(s);
  ```

  - 아래와 같이 변경

  ```
  Function<String, Integer> f = Integer::parseInt; // 메서드 참조
  ```

- 위 메서드 참조에서 람다식의 일부가 생략되었지만, 컴파일러는 생략된 부분을 우변의 parseInt메서드의 선언부로부터, 또는 좌변의 Function인터페이스에 지정된 지네릭 타입으로부터 쉽게 알아낼 수 있다.
- 한 가지 예를 더 보자. 아래의 람다식을 메서드 참조로 변경한다면, 어떻게 되겠는가? 람다식에서 생략해도 좋을 만한 부분이 어디인지 한번 생각해 보자.

```
BiFunction<String, String, Boolean> f = (s1, s2) -> s1.equals(s2);
```

- 참조변수 f의 타입만 봐도 람다식이 두 개의 String타입의 매개변수를 받는다는 것을 알 수 있으므로, 람다식의 매개변수들은 없어도 된다. 위의 람다식에서 매개변수들을 제거해서 메서드 참조로 변경하면 아래와 같다.

  - 즉, 아래의 식에서.

  ```
  BiFunction<String, String, Boolean> f = (s1, s2) -> s1.equals(s2);
  ```

  - 아래의 식으로 변경.

  ```
  BiFunction<String, String, Boolean> f = String::equals; // 메서드 참조
  ```

- 매개변수 s1과 s2을 생략해버리고 나면 equals만 남는데, 두 개의 String을 받아서 Boolean을 반환하는 equals라는 이름의 메서드는 다른 클래스에도 존재할 수도 있기 때문에 equals 앞에 클래스 이름은(여기서는 String) 반드시 필요하다.
- 메서드 참조를 사용할 수 있는 경우가 한 가지 더 있는데, 이미 생성된 객체의 메서드를 람다식에서 사용한 경우에는 클래스 이름 대신 그 객체의 참조변수를 적어줘야 한다.

```
MyClass obj = new MyClass();
Function<String, Boolean> f = (x) -> obj.equals(x); // 람다식
Function<String, Boolean> f2 = obj::equals; // 메서드 참조
```

- 지금까지 3가지 경우의 메서드 참조에 대해서 알아봤는데, 정리하면 다음과 같다.
  (람다식을 메서드 참조로 변환하는 방법)

| 종류                          | 람다                       | 메서드 참조       |
| ----------------------------- | -------------------------- | ----------------- |
| static메서드 참조             | (x) -> ClassName.method(x) | ClassName::method |
| 인스턴스메서드 참조           | (obj, x) -> obj.method(x)  | className::method |
| 특정 객체 인스턴스메서드 참조 | (x) -> obj.method(x)       | obj::method       |

- 정리
  - `하나의 메서드만 호출하는 람다식은 '클래스이름::메서드이름' 또는 '참조변수::메서드이름'으로 바꿀 수 있다.`

## 생성자의 메서드 참조

- 생성자를 호출하는 람다식도 메서드 참조로 변환할 수 있다.

```
Supplier<MyClass> s = () -> new MyClass(); // 람다식
Supplier<MyClass> s = Myclass:new; // 메서드 참조
```

- 매개변수가 있는 생성자라면, 매개변수의 개수에 따라 알맞은 함수형 인터페이스를 사용하면 된다. 필요하다면 함수형 인터페이스를 새로 정의해야 한다.

```
Function<Integer, MyClass> f = (i) -> new MyClass(i); // 람다식
Function<Integer, Myclass> f2 = MyClass::new; // 메서드 참조

BiFunction<Integer, String, MyClass> bf = (i, s) -> new MyClass(i, s);
BiFunction<Integer, String, Myclass> bf2 = MyClass::new; // 메서드 참조
```

- 그리고 배열을 생성할 때는 아래와 같이 하면 된다.

```
Function<Integer, int[]> f = x -> new int[x]; // 람다식
Function<Integer, int[]> f2 = int[]::new; // 메서드 참조
```

- 메서드 참조는 람다식을 마치 static변수처럼 다룰 수 있게 해준다. 메서드 참조는 코드를 간략히 하는데 유용해서 많이 사용된다. 람다식을 메서드 참조로 변환하는 연습을 많이해서 빨리 익숙해지기 바란다.

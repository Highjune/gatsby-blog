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

| 함수형 인터페이스   | 메서드 | 설명 |
| ------------------- | ------ | ---- |
| BiConsumer`<T,U>`   | d      | d    |
| BiPredicate`<T,U>`  | d      | d    |
| BiFunction`<T,U,R>` | d      | d    |

# 5. Function의 합성과 Predicate의 결합

# 6. 메서드 참조

#

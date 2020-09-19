---
title: '지네릭스'
date: 2020-08-29 20:43:00
category: 'java'
draft: false
---

#### 자바의 정석을 공부하면서 정리한 것

# 지네릭스(Generics)

- 컴파일시 타입을 체크해 주는 기능(compile-time type check) - JDK1.5
- 객체의 타입 안정성을 높이고 형변환의 번거로움을 줄여줌(코드 간결)
- 기존에도 컴파일시 체크해주긴 했는데 한계가 존재했다.

- 한계란?

  ```
  package test;

  import java.util.ArrayList;

  public class GenericTest {
      public static void main(String[] args) {
          ArrayList list = new ArrayList();

          list.add(10);
          list.add(20);
          list.add("30");

          Integer i = (Integer)list.get(2); // 컴파일은 OK, 그러나 실행시에 에러가 발생한다.

          System.out.println(list);
      }
  }
  ```

  - 위의 코드는 컴파일은 OK이지만, `ClassCastException(형변환)` 실행에러가 난다. 왜냐하면 list.get()의 모든 타입은 Object 라서 (Integer)로 형변환이 가능하다는 로직 때문에 컴파일이 가능하다. 하지만 실제로는 list.get(2)의 형은 "30", 즉 `String`이므로 `Integer`로 형변환이 불가능한다. 이 불가능한 것을 컴파일러는 체크하지 못하므로 한계.
  - 실행시 에러가 발생하면 프로그램이 off된다. 그래서 실행시 에러보다 컴파일러 에러가 그나마 낫다. 따라서 컴파일에서 애초에 그 에러를 잡는 것이 중요하다.
  - 실행시 에러를 컴파일 에러로 들고 오기 위한 고민이 바로 `Generics`
  - 따라서 아래와 같이 변경하면 가능.

  ```
  package test;

  import java.util.ArrayList;

  public class GenericTest {
      public static void main(String[] args) {
          ArrayList<Integer> list = new ArrayList<Integer>();

          list.add(10);
          list.add(20);
          list.add(30); //지네릭스 덕분에 타입 체크가 강화됨

          Integer i = list.get(2); //형변환이 필요없다. 애초에 Integer밖에 못 들어오는 것을 알고 있기에

          System.out.println(list);
      }
  }
  ```

  - 만약 여러 종류를 저장하고 싶다면 `<Object>` 옵션을 주면 된다. 애초에 ArrayList는 Object를 넣기 때문에 안 넣어도 되지 않나? 아니다. JDK 1.5부터는 명시해야 한다.

  ```
  ArrayList<Object> list = new ArrayList<Object>();
  ```

- 클래스 안에 Object 타입이 있는 것들은 `일반클래스` -> `지네릭클래스`로 변경되었다.

  - 클래스 옆에 문자가 있는 것들(타입변수, `<E>`)은 지네릭클래스다.
  - ex) ArrayList -> ArrayList`<E>`

  ```
  ArrayList<Tv> tvList = new ArrayList<Tv>();

  tvList.add(new Tv()) //ok
  tvList.add(new Audio()) // 컴파일 에러. Tv 외에 다른 타입은 저장 불가
  ```

## 타입 변수

- 클래스 작성시, Object타입 대신 타입 변수(E)를 선언해서 사용
- E는 ELement(요소)의 약자. 타입 변수는 `<E>` 이외에 다른 것들 `<T>`, `<X>`, `<AB>` 등 여러가지 다 써도 된다.
- 객체를 생성시, 타입 변수`<E>` 대신 실제 타입 `<Tv>`을 지정(대입)

`ArrayList<Tv> tvList = new ArrayList<Tv>();`

- 위에서 앞과 뒤의 타입변수 `<Tv>` 는 일치해야 한다.

```
public class ArrayList<E> extends AbstractList<E>{
    private transient E[] elementData;
    public boolean add(E o) { /* 내용생략 */}
    public E get(int index) { /* 내용생략 */}
    ...
}
```

- 아래와 같이 변환된다.

```
public class ArrayList extends AbstractList<Tv>{
    private transient Tv[] elementData;
    public boolean add(Tv o) { /* 내용생략 */}
    public Tv get(int index) { /* 내용생략 */}
    ...
```

- 타입 변수 대신 실제 타입이 지정되면, 형변환 생략가능

## 지네릭스 용어

- Box`<T>`
  - 지네릭 클래스, `T의 Box` 또는 `T Box`라고 읽는다.
- T
  - 타입 변수 또는 타입 매개변수(T는 타입 문자)
  - 실제 타입을 정해준다. ex) String 등
- Box

  - 원시 타입(raw type), 일반 클래스

- ex1) class Box`<T>`{ }
  - Box`<T>` - 지네릭클래스
  - Box - 원시타입
  - T - 타입변수
- ex2)
  ```
  Box<Stirng> b = new Box<String>();
  ```
  - `<String>` 은 대입된 타입(매개변수화된 타입, parameterized type). 참조변수의 타입과 생성자의 타입 `<String>`은 일치되어야 한다.
  - Box`<String>` 는 지네릭 타입 호출

## 지네릭 타입과 다형성

- 참조 변수와 생성자의 대입된 타입은 일치해야 한다.

  ```
  (클래스 관계)
  class Product {}
  class Tv extends Product {}
  class Audio extends Product {}
  ```

  ```
  ArrayList<Tv> list = new ArrayList<Tv>();      //OK. 일치
  ArrayList<Product> list = new ArrayList<Tv>(); //에러, 불일치(다형성 성립 안됨)
  ```

  ```
  public static void printAll(ArrayList<Product> list) {
      for(Product p : list)
          System.out.println(p);
  }

  (메인함수)
  public static void main(String[] args) {
      printAll(tvList); //컴파일 에러가 난다
  }
  ```

  - 바로 위 코드에서 에러나는 이유는, printAll의 함수의 파라미터는 product타입을 받아야 하는데, tv타입을 대입했으므로.

* 지네릭 클래스간의 다형성은 성립(여전히 대입된 타입은 일치해야 한다)

```
List<Tv> list = new ArrayList<Tv>(); //OK. 다형성. ArrayList가 List를 구현
List<Tv> list = new LinkedList<Tv>();   //OK. 다형성. LinkedList가 List를 구현
```

- 매개변수의 다형성도 성립

  - 왜냐하면
    `boolean add(E e) {...}` -> `boolean add(Product e) {...}` 로 변환되기 때문. 매개변수에는 Product와 그 자손들이 다 들어갈 수 있다.

  ```
  ArrayList<Product> list = new ArrayList<Product>();

  list.add(new Product());
  list.add(new Tv());     //OK, 하지만 list.get(1)로 꺼낼 때는 타입이 Product이다.
  list.add(new Audio());  //OK

  Product p = list.get(0); // OK. 형변환이 필요없음
  Tv t = (Tv)list.get(1); // (Tv)로 형변환 해줘야 한다.
  ```

  - 위의 코드 중 제일 마지막에 `Tv t = (Tv)List.get(1)`에서 (Tv)로 형변환 하는 이유는?
    `E get(int index) {...}` -> `Product get(int index) {...}` 이므로 get의 타입은 Product 이기 때문에

## Iterator`<E>`

- 클래스를 작성할 때, Object타입 대신 T와 같은 타입 변수를 사용

  - 예전에는 아래와 같았지만,

  ```
  public interface Iterator {
    boolean hasNexT();
    Object nexT(); //Object
    void remove();
  }

  Iterator it = list.iterator();
  while(it.hasNext()){
    Student s = (Student)it.next(); //형변환 필요
    ...
  }
  ```

  - 지네릭스를 사용하면 형변환이 필요가 없다.(코드 간결)

  ```
  public interface Iterator<E>{
    boolean hasNexT();
    E nexT(); // next()가 Student 타입
    void remove();
  }

  Iterator<Student> it = list.iterator(); // 타입 명시
  while(it.hasNext()){
    Student s = (Student)it.next(); //형변환 필요 X
    ...
  }

  ```

## HashMap<K,V>

- 여러 개의 타입 변수가 필요한 경우, 콤마(,)를 구분자로 선언.
- Key의 K, Value의 Value이다. 꼭 K와 V를 써야 하는 것은 아니다.

```
HashMap<String, Student> map = new HashMap<String, Student>(); // 생성
map.put("자바왕", new Student("자바왕", 1, 1, 100, 100, 100)); // 데이터 저장
```

- 위의 코드로 인해 밑의 코드가,

```
public class HashMap<K, V> extends AbstractMap<K, V> {
  ...
  public V get(Object key) { 내용생략 }
  public V put(K key, V value) { 내용생략 }
  public V get(Object key) { 내용생략 }
  ...
}
```

- 다음과 같이 바뀐다.

```
public class HashMap extends AbstractMap {
  ...
  public Student get(Object key) { 내용생략 }
  public Student put(Student key, Student value) { 내용생략 }
  public Student get(Object key) { 내용생략 }
  ...
}
```

- 그래서 `Student s1 = (Student)map.get("1-1");` 가 `Student s1 = map.get("1-1");` 로 가능하다(형 변환 필요없음)

- JDK 1.7부터 참조 변수와 생성자의 대입된 타입이 일치할 경우(그래야만 하고) 생성자에 타입지정 생략가능

```
HashMap<String, Student> map = new HashMap<>();
```

참고 : 자바의 정석 - 남궁성

## 지네릭 타입의 형변환

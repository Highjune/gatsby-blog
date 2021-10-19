---
title: '지네릭스'
date: 2020-08-29 20:43:00
category: 'java'
draft: false
---

- 추가 참고글
  - [st-lab.tistory](https://st-lab.tistory.com/153)

#### 자바의 정석을 공부하면서 정리한 것

# 지네릭스(Generics)

- 컴파일시 타입을 체크해 주는 기능(compile-time type check) - JDK1.5부터
  - 컴파일러에게 타입을 알려줌
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
          list.add("30"); //사실 숫자만 넣고 싶었는데 String 넣었음에도 구분못함

          Integer i = (Integer)list.get(2); // 컴파일은 OK, 그러나 실행시에 에러가 발생한다.

          System.out.println(list);
      }
  }
  ```

  - 위의 코드는 컴파일은 OK이지만, `ClassCastException(형변환)` 실행에러가 난다. 왜냐하면 list.get()의 모든 타입은 Object 라서 (Integer)로 형변환이 가능하다는 로직 때문에(Object->String 형변환 가능) 컴파일이 가능하다. 하지만 실제로는 list.get(2)의 형은 "30", 즉 `String`이므로 `Integer`로 형변환이 불가능하므로 에러 발생. 컴파일러는 실제로 안에 뭐가(String, Interger 등) 들어있는지 체크하지 못하므로 한계.
  - 실행시 에러가 발생하면 프로그램이 off된다. 그래서 실행시 에러보다 컴파일러 에러가 그나마 낫다. 따라서 컴파일에서 애초에 그 에러를 잡는 것이 중요하다.
  - 실행시 에러(RuntimeException)를 컴파일 에러(Compile Error)로 들고 오기 위한 고민이 바로 `Generics`. 컴파일러에게 타입 정보면 제공해주면 가능.
  - 따라서 아래와 같이 변경하면 가능.

  ```
  package test;

  import java.util.ArrayList;

  public class GenericTest {
      public static void main(String[] args) {
          ArrayList<Integer> list = new ArrayList<Integer>();

          list.add(10);
          list.add(20);
          //list.add("30"); // 컴파일러가 에러를 잡아준다. String은 들어갈 수 없음
          list.add(30); //지네릭스 덕분에 타입 체크가 강화됨

          Integer i = list.get(2); //원래는 list.get()의 리턴이 Object라서 형 변환해줘야 하지만. 형변환도 생략가능. 애초에 Integer밖에 못 들어오는 것을 알고 있기에.

          System.out.println(list);
      }
  }
  ```

  - 만약 여러 종류를 저장하고 싶다면 `<Object>` 옵션을 주면 된다. 애초에 ArrayList는 Object를 넣기 때문에 안 넣어도 되지 않나? 아니다. JDK 1.5부터는 반드시 명시해야 한다.

  ```
  ArrayList<Object> list = new ArrayList<Object>();
  ```

- 클래스 안에 Object 타입이 있는 것들은 `일반클래스` -> `지네릭클래스`로 변경되었다. 지네릭 타입을 반드시 써줘야 하는 클래스들이 있다. 타입을 꼭 지정해 줘야 한다.

  - 클래스 이름 옆에 문자가 있는 것들(타입변수, `<E>`)은 지네릭클래스다.
  - ArrayList 클래스 들어가서 설명 보면 알 수 있다.
  - ex) ArrayList -> ArrayList`<E>`

  ```
  ArrayList<Tv> tvList = new ArrayList<Tv>();

  tvList.add(new Tv()) //ok
  tvList.add(new Audio()) // 컴파일 에러. Tv 외에 다른 타입은 저장 불가
  ```

## 타입 변수

- 클래스 작성시, Object타입 대신 타입 변수`<E>`를 선언해서 사용
- E는 Element(요소)의 약자. T는 Type의 약자. 타입 변수는 `<E>`, `<T>` 이외에 다른 것들 `<X>`, `<AB>` 등 여러가지 다 써도 된다.
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
  - 원래는 아래와 같았다.
  ```
  ArrayList tvList = new ArrayList();
  tvList.add(new Tv());
  Tv t = (Tv)tvList.get(0);  // 꺼낼 때의 타입이 Object이므로
  ```
  - 하지만, 아래와 같이 형변환 생략가능
  ```
  ArrayList<Tv> tvList = new ArrayList<Tv>();
  tvList.add(new Tv());
  Tv t = tvList.get(0);
  ```

## 지네릭스 용어

- Box`<T>`
  - 지네릭 클래스, `T의 Box` 또는 `T Box`라고 읽는다.
- T
  - 타입 변수 또는 타입 매개변수(T는 타입 문자)
  - 실제 타입을 정해준다. ex) String 등
- Box

  - 원시 타입(raw type), 일반 클래스(지네릭 클래스로 바뀌기 전 타입)
  - ex) Array`<String>` 이 되기 전의 그냥 Array

- ex1) class Box`<T>`{ }
  - Box`<T>` - 지네릭클래스
  - Box - 원시타입
  - T - 타입변수
- ex2)
  ```
  Box<Stirng> b = new Box<String>();
  ```
  - `<String>` 은 대입된 타입(매개변수화된 타입, parameterized type). 참조변수의 타입과 생성자의 타입 `<String>`은 일치되어야 한다.
  - 생성할 때마다 `<String>` 말고 다른 타입을 적용해도 된다.
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
  ArrayList<Product> list = new ArrayList<Tv>(); //에러, 불일치(다형성 성립 안됨) 즉 부모 - 조상 관계라도 안된다
  ```

  ```
  public static void printAll(ArrayList<Product> list) {
      for(Product p : list)
          System.out.println(p);
  }

  (메인함수)
  public static void main(String[] args) {
      ArrayList<Product> productList = new ArrayList<Product>();
      ArrayList<Tv> tvList = new ArrayList<Tv>();
      List<Tv> tvList2 = new ArrayList<Tv>(); // OK. 다형성

      // public boolean add(Product e) {}로 변환됨
      productList.add(new Tv());
      productList.add(new Audio()); // Audio가 Product의 자손이므로 가능

      // public boolean add(Tv e) {}로 변환됨
      tvList.add(new Tv());
      tvList.add(new Audio()); // 안됨. Tv와 Audio는 아무 관계 없으므로

      printAll(tvList); //컴파일 에러가 난다
  }
  ```

  - 바로 위 코드에서 에러나는 이유는, printAll의 함수의 파라미터는 product타입을 받아야 하는데, tv타입을 대입했으므로.

- 지네릭 클래스간의 다형성은 성립(여전히 대입된 타입은 일치해야 한다)

```
List<Tv> list = new ArrayList<Tv>();    //OK. 다형성. ArrayList가 List를 구현
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
    `E get(int index) {...}` -> `Product get(int index) {...}` 이므로 get의 타입은 Object가 아니라 Product 이기 때문에

## Iterator`<E>`

- 지네릭 클래스 중 1개

- 클래스를 작성할 때, Object타입 대신 T와 같은 타입 변수를 사용

  - 예전에는 아래와 같았다.

  ```
  public interface Iterator {
    boolean hasNext();
    Object next(); //next()의 리턴이 Object 타입
    void remove();
  }

  Iterator it = list.iterator();
  while(it.hasNext()){
    Student s = (Student)it.next(); //형변환 필요
    ...
  }
  ```

  - 하지만 지네릭스를 사용하면 형변환이 필요가 없어져서 아래와 같이 코드 간결해진다.

  ```
  public interface Iterator<E>{
    boolean hasNext();
    E next(); // next()의 리턴이 Student 타입
    void remove();
  }

  Iterator<Student> it = list.iterator(); // 타입 명시
  while(it.hasNext()){
    Student s = (Student)it.next(); //형변환 필요 X
    ...
  }

  ```

  - ex)

  ```
    public class test {
    public static void main(String[] args) {
      ArrayList<Student> list = new ArrayList<Student>();
      list.add(new Student("자바왕", 1, 1));
      list.add(new Student("자바짱", 1, 2));
      list.add(new Student("홍길동", 3, 4));

      Iterator<Student> it = list.iterator();
      //Iterator it = list.iterator();
      while(it.hasNext()) {
        // System.out.println((Student).it.next()).name);
        System.out.println(it.next()).name); // 훨씬 간결
      }
    }
  }

  ```

## HashMap<K,V>

- 여러 개의 타입 변수가 필요한 경우, 콤마(,)를 구분자로 선언. (3개 이상도 가능)
- Key의 K, Value의 Value이다. 꼭 K와 V를 써야 하는 것은 아니다.

```
HashMap<String, Student> map = new HashMap<String, Student>(); // 생성
map.put("자바왕", new Student("자바왕", 1, 1, 100, 100, 100)); // 데이터 저장
```

- HashMap을 HashMap<String, Student>로 변환하게 되면 밑의 코드가,

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

## 제한된 지네릭 클래스

- extends로 대입할 수 있는 타입을 제한

```
class FruitBox<T extends Fruit>{ // Fruit의 자손만 타입으로 지정가능
	ArrayList<T> list = new ArrayList<T>();

}

public class Test {
	public static void main(String[] args) {
		FruitBox<Apple> appleBox = new FruitBox<Apple>(); //OK
		FruitBox<Toy> toyBox = new FruitBox<Toy>(); //에러. Toy는 Fruit의 자손이 아님
	}
}

```

- 인터페이스인 경우에도 extends를 사용(`implements가 아니다`)

```
interface Eatable {}
class FruitBox<T extends Eatable> {...} // Eatable도 인터페이스

```

- ex)

```
interface Eatable {}

class Fruit implements Eatable {
	public String toString() {
		return "Fruit";
	}
}

class Apple extends Fruit {
	public String toString()  { return "Apple";}
}
class Grape extends Fruit {
	public String toString()  { return "Grape";}
}
class Toy {
	public String toString()  { return "Toy";}
}



public class Test {
	public static void main(String[] args) {
		FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
		FruitBox<Apple> appleBox = new FruitBox<Apple>();
		FruitBox<Grape> grapeBox = new FruitBox<Grape>();
//		FruitBox<Grape> grapeBox = new FruitBox<Apple>(); //에러, 앞 뒤 타입 불일치
//		FruitBox<Toy> toyBox = new FruitBox<Toy>(); // 에러, FruitBox는 Fruit를 상속받아야 하며 Eatable를 구현한 것만 들어올 수 있다. Toy는 해당 안됨
    Box<Toy> toyBox2 = new Box<Toy>(); // OK

		fruitBox.add(new Fruit()); //Box클래스에서 add(T item)메서드가 add(Fruit item) 으로 변경되기 때문에 다형성에 의해 Fruit포함 자손들 다 add 매개변수로 들어올 수 있다.
		fruitBox.add(new Apple());
		fruitBox.add(new Grape());
		appleBox.add(new Apple());
//		appleBox.add(new Grape()); //에러. Grape는 Apple의 자손이 아님
		grapeBox.add(new Grape());

		System.out.println("fruitBox" + fruitBox);
	}
}

class FruitBox<T extends Fruit & Eatable> extends Box<T> {} //클래스(Fruit)와 인터페이스(Eatable) 둘 다 구현할 때는 , 로 연결하면 안되고 &로 구분해야 한다.
//사실 위는 class FruitBox<T extends Fruit> extends Box<T> {} 와 똑같다. Fruit이 이미 Eatable을 구현했으므로 Fruit만 상속받더라도 Eatable을 구현한 것과 같은 결과

class Box<T> {
	ArrayList<T> list = new ArrayList<T>(); // item을 저장할 list
	void add(T item) {list.add(item);} //박스에 item을 추가
	T get(int i) {return list.get(i);} //박스에서 item을 꺼낼 때
	int size() {return list.size();}
	public String toString() {return list.toString();}
}

```

## 지네릭스의 제약

- 타입 변수에 대입은 인스턴스별로 다르게 가능하다

```
class Box<T> {}
Box<Apple> appleBox = new Box<Apple>(); //OK. Apple객체만 저장가능
Box<Grape> grapebox = new Box<Grape>(); //OK. Apple객체만 저장가능
```

제약1. static멤버에 타입 변수 사용 불가

- static 멤버는 모든 인스턴스에 공통이므로 불가.(인스턴스별로 타입 변수 지정이 가능해야하는데)

```
class Box<T> {
  static T item; // 에러
  static int compare(T t1, T t2) {...} // 에러
  ...
}
```

제약2. 객체 생성할 때(배열 생성할 때 등) 타입 변수 사용불가. 타입 변수로 배열 선언은 가능. new 연산자 뒤에 T못 쓴다. (new 연산자는 뒤의 타입이 확정되어 있어야 하는데, new 연산자 뒤에 T를 쓰면 뭐가 오는지 확실하게 모르므로)

```
class Box<T>{
  T[] itemArr; //지네릭 배열 선언은 OK. T타입의 배열을 위한 참조변수
    ...
  T[] toArray() {
    T[] tmpArr = new T[itemArr.length]; // 에러. 지네릭 배열 생성불가, new T 자체가 불가하다.
    ...
  }
}
```

## 와일드 카드 <?>

- 하나의 참조 변수로 대입된 타입이 다른 객체를 참조 가능
- 원래 지네릭스에서는 참조변수와 생성자에 대입된 타입이 일치해야 하는데, 다형성처럼 하나의 참조변수로 여러 객체를 가리키고자 만들어진 것이 와일드 카드

- 와일드 카드를 사용하면 참조변수의 타입과 생성자의 타입이 불일치 해도 가능하다
  - list 참조변수 하나로 Tv와 Audio 둘 모두 가리킬 수 있다.(Tv와 Audio는 모두 Product의 자식 클래스)

```
ArrayList<? extends Product> list = new ArrayList<Tv>(); // OK
ArrayList<? extends Product> list = new ArrayList<Audio>(); // OK
```

- 와일드 카드의 3가지 용법

  1. `<? extends T>` 와일드 카드의 상한 제한. T와 그 자손들만 가능(가장 많이 쓰임)
  2. `<? super T>` 와일드 카드의 하한 제한. T와 그 자손들만 가능
  3. `<?>` 제한없음. 모든 타입이 가능 <? extends Object>와 동일

- 메서드의 매개변수에 와일드 카드 사용가능

```
static Juice makeJuice(FruitBox<? extends Fruit> box) {
  String tmp : "";
  for(Fruit f : box.getList()) tmp += f + " ";
  return new Juice(tmp);
}

..(중략)..

//아래 둘 다 가능. 만약 makeJuice(FruitBox<Fruit> box)로 정의되어 있으면 매개변수로 new FruitBox<Fruit>() 만 들어갈 수 있다. (일치하는 것만 가능). 즉, 매개변수 FruitBox<? etends Fruit> box 는 FruitBox<Fruit>와 Fruitbox<Apple> 2가지 모두를 포함한다.(자신 포함 자식들도 들어갈 수 있으니)

System.out.println(Juicer.makeJuice(new FruitBox<Fruit>()));
System.out.println(Juicer.makeJuice(new FruitBox<Apple>()));
```

## 지네릭 메서드

- 지네릭 타입이 선언된 메서드(타입 변수는 메서드 내에서만 유효)

```
static <T> void Sort(List<T> list, Comparator<? super T> c)
```

- 클래스의 타입 매개변수`<T>`와 메서드의 타입 매개변수 `<T>`는 별개
  - 밑에서 클래스의 FruitBOx`<T>`와 메서드에 있는 `<T>`는 별개다. 전자에 String, 후자에 Integer로 따로 설정 가능

```
class FruitBox<T> {
  ...
  static <T> void sort(List<T> list, Comparator<? super T> c){
    ...
  }
}
```

- 메서드를 호출할 때마다 타입을 대입해야(대부분 생략 가능)
  - 지네릭 클래스는 객체를 생성할 때 지네릭 타입을 대입하는 반면, 메서드는 호출할 때마다.

```
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Fruit> fruitBox = new FruitBox<Apple>();
  ...
System.out.println(Juicer.<Fruit>makeJuice(fruitBox)); //여기서 <Fruit>은 생략가능. 위의 new FruitBox<Fruit>(); 에서 이미 <Fruit> 이라고 이미 명시했기 때문에.
System.out.println(Juicer.<Apple>makeJuice(appleBox));//여기서 <Apple>은 생략가능. 위의 new FruitBox<Apple>>(); 에서 이미 <Apple> 이라고 이미 명시했기 때문에.
...

//밑의 파라미터 부분의 FruitBox<T>를 정의하는 것이 static 바로 뒤에 <T extends Fruit>임
static <T extends Fruit> Juice makeJuice(FruitBox<T> box){
  String tmp : "";
  for(Fruit f : box.getList()) tmp += f + " ";
  return new Juice(tmp);
}

```

- 메서드를 호출할 때 타입을 생략하지 않을 때는 클래스 이름 생략 불가(에러가 날시에만 명시해주면 되는데, 사실 거의 대부분 생략가능)

```
System.out.println(<Fruit>makeJuice(fruitBox)); //에러, 클래스 이름 생략불가
System.out.println(this.<Fruit>makeJuice(fruitBox)); // OK
System.out.println(Juicer.<Fruit>makeJuice(fruitBox)); // OK
```

- 지네릭 메서드 vs 와일드카드 메서드
  - `지네릭 메서드`는 지네릭 클래스처럼 호출할 때마다 다른 타입을 대입할 수 있는 것이며, `와일드카드`는 하나의 참조변수로 대입된 타입이 다른 여러 지네릭 객체를 다룰 수 있게 하기 위한 것. (용도가 조금 다르다)
  - 와일드 카드를 쓰지 못하는 경우가 있는데, 그럴 때 지네릭 메서드를 사용한다.
  - 아래 2개는 같이 쓸 수 있다.
    - 지네릭 메서드
    ```
    static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
      String tmp : "";
      for(Fruit f : box.getList()) tmp += f + " ";
      return new Juice(tmp);
    }
    ```
    - 와일드카드 메서드
    ```
    static Juice makeJuice(FruitBox<? extends Fruit> box) {
      String tmp : "";
      for(Fruit f : box.getList()) tmp += f + " ";
      return new Juice(tmp);
    }
    ```

## 지네릭 타입의 형변환

- 지네릭 타입과 원시 타입 간의 형변환은 바람직하지 않다.(경고 발생)
  - 즉 `class Box<T> {}`로 선언이 되어있는데 타입을 대입하지 않고 Box만 쓰면 안된다라는 것. 원시타입과 지네릭 타입을 섞어서 쓰지 말기
  - 여기서 원시타입을 쓰는 것을 비추천

```
Box<Object> objBox = null;
Box box = (Box)objBox;  // OK. 지네릭 타입(Box<Object>) -> 원시 타입(Box). 가능하지만 경고 발생
objBox = (Box<Object>)box; // OK. 원시타입(Box) -> 지네릭 타입(Box<Object>). 가능하지만 경고 발생
```

- ex)

```
Box b = null;
Box<String> bStr = null;

b = (Box)bStr; // Box<String> -> Box 가능, 그러나 경고
bStr = (Box<String>)b; // Box -> Box<String> 가능, 그러나 경고
```

- 위의 예제는 아래와 같이 사실 원시타입으로 지네릭 타입을 가리킨다는 것과 동일하다는 것.
  `Box b = new Box<String>();` 이 가능. 왜냐하면 `Box b = (Box)new Box<String>();`처럼 (Box)가 생략이 되어 있기 때문에. b는 대입된 타입이 없는 원시타입이기 때문에 `b.add(new Integer(100));` 가능하다. 그래서 `Box<String> b = new Box<String>();` 으로 수정하면 `b.add(new Integer(100));`이 에러가 뜬다.

- 서로 다른 타입이 대입된 지네릭 타입끼리는 서로 형변환이 안 된다.

```
Box<Object> objBox = null;
Box<String> bStr = null;

objBox = (Box<Object>)strBox; // 에러. Box<String> -> Box<Object> 안됨
strBox = (Box<String>)objBox; // 에러. Box<Object> -> Box<String> 안됨
```

- 즉, `Box<String> b = new Box<Object>();`이 안된다는 말은 `Box<String> b = (Box<String>)new Box<Object>();` 처럼 형변환이 안된다는 말이다.

- 와일드 카드가 사용된 지네릭 타입으로는 형변환 가능

```
Box<Object>objBox = (Box<Object>)new Box<String>(); // 에러, 형변환 불가능
Box<? extends Ojbect> wBox = (Box<? extends Object>)new Box<String>(); // OK, 형변환 가능
Box<? extends Object> wBox = new Box<String>(); //OK, 위 문장과 동일(형변환이 생략되어 있는 것임). String은 Object의 자손이므로 가능
```

- ex)

```
FruitBox<? extends Fruit> abox = new FruitBox<Apple>();
FruitBox<Apple> appleBox = (FruitBox<Apple>)aBox; // OK. 경고발생
```

## 지네릭 타입의 제거

- 컴파일러는 지네릭 타입을 제거하고, 필요한 곳에 형변환을 넣는다. (안정적인 하위호환성 때문에. JDK 1.5 이전버전과 문제 없이 돌아가게 하려고)

1. 지네릭 타입의 경계(bound)를 제거

   ```
   class Box<T>{
     void add(T t) {
       ...
     }
   }
   ```

   - 컴파일이 되면 위가 아래처럼 된다.

   ```
   class Box{
     void add(Object obj){
       ...
     }
   }
   ```

   또,

   ```
   class Box<T extends Fruit>{
     void add(T t) {
       ...
     }
   }
   ```

   - 컴파일이 되면 위가 아래처럼 된다.

   ```
   class Box{
     void add(Fruit t){
       ...
     }
   }
   ```

2. 지네릭 타입 제거 후에 타입이 불일치하면, 형변환을 추가

- 지네릭의 장점 중 하나가 형변환을 생략하는 것이었었음.
- ex1)

```
T get(int i) {
  return list.get(i); // (Fruit)list.get(i)처럼 타입 T(ex. Fruit)로 형변환이 생략되어 있음
}
```

- 위의 코드는 아래처럼 변경(그래서 아래에서 `T get(int i)`에서의 T가 컴파일에 의해서 Fruit으로 변환)

```
Fruit get(int i) {
  return (Fruit)list.get(i);
}
```

- ex2)

```
static Juice makeJuice(FruitBox<? extends Fruit> box){
  String tmp = "";
  for(Fruit f : box.getList()) // box.getList()가 Fruit과 그 자손이므로 형변환 필요x(와일드 카드 덕분)
    tmp += f + " ";
}
```

- 하지만 위의 코드에 비해 아래는 일일이 형변환 해줘야 한다.

```
stataic Juice makeJuice(FruitBox box){
  String tmp = "";
  Iterator it = box.getList().iterator();
  while(it.hasNext()){
    tmp += (Fruit)it.next() + " "; // 형변환 필요
  }
}
```

참고 : 자바의 정석 - 남궁성

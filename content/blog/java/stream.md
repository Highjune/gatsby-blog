---
title: 'Stream'
date: 2022-09-07 10:48:00
category: 'java'
draft: false
---

#### 자바의 정석 기초편(유투브 강의)

# 스트림(Stream)

## 다양한 데이터 소스를 표준화된 방법으로 다루기 위한 것

- 데이터 소스란? 컬렉션, 배열 같은 것
- 표준화된 방법? 이제까지는 Collection FrameWork(List, Set, Map)을 이용해서 다뤄온 것
  - 하지만 반쪽짜리 표준화다. 왜냐하면 list, set은 map과 사용방법이 다르기 때문. 표준화에 실패
- 하지만 jdk 1.8 부터 stream을 통해서 정말 다양한 데이터 소스를 표준화된 방법으로 다루는 것이 가능

- 컬렉션(List, Set, Map) 과 배열을 다 Stream으로 변환할 수 있다.

  - 컬렉션, 배열 -> Stream -> (중간연산) -> (최종연산) 결과.
    - 중간연산은 n 번 가능, 최종연산은 1번만 가능.
    - stream이 나오기 전에는 (중간연산 -> 최종연산) 이 다 달랐다. 하지만 다 stream으로 변환할 수 있으므로 통일 가능.
    - 그래서 스트림으로 작업을 처리할 때는 모든 것들을 우선 1. 스트림으로 만든 후 2. 중간연산(0 ~ n번) 3. 최종연산(0~1번)으로 결과를 도출.

- stream으로 변환(생성하는 여러 방법들)

```
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

Stream<Integer> intStream1 = list.stream(); // 컬렉션, Collections.stream() 이라는 메서드 있음.(Stream 반환)
Stream<String> strStream = Stream.of(new String[]{"a", "b", "c"}); // 배열
Stream<Integer> evenStream = Stream.iterate(0, n->n+2); // 0, 2, 4, 6, ...
Stream<Double> randomStream = Stream.generate(Math::random); // 람다식
IntStream intStream2 = new Random().ints(5); // 난수 스트림(크기가 5);
```

## 스트림이 제공하는 기능 - 중간 연산과 최종 연산

- 중간 연산
  - 연산결과가 스트림인 연산. 반복적으로 적용가능
- 최종 연산

  - 연산결과가 스트림이 아닌 연산. 단 한번만 적용가능(스트림의 요소를 소모)

- ex)

```
stream.distinct().limit(5).sorted().forEach(System.out::println)
```

  - distinct()
    - 중간연산(중복제거)
  - limit(5)
    - 중간연산(5개 자르기)
  - sorted()
    - 중간연산(정렬)
  - forEach(System.out::println)
    - 최종연산(출력)

```
String[] strArr = {"dd", "aaa", "CC", "cc", "b"};

Stream<String> stream = Stream.of(strArr); // 문자열 배열이 소스인 스트림
Stream<String> filteredStream = stream.filter(); // 걸러내기 (중간 연산)
Stream<String> distinctedStream = stream.distinct(); // 중복제거 (중간 연산)
Stream<String> sortedStream = stream.sort(); // 정렬 (중간 연산)
Stream<String> limitedStream = stream.limit(5); // 스트림 자르기 (중간 연산)
int total = stream.count(); // 요소 개수 세기(최종연산)
```

## 스트림의 특징

1. 스트림은 데이터 소스(원본)으로부터 데이터를 읽기만할 뿐 변경하지 않는다.

```
List<Integer> list = Arrays.asList(3, 1, 5, 4, 2);
List<Integer> sortedList = list.stream().sorted() // list를 정렬해서
                            .collect(Collectors.toList()); // 새로운 List에 저장
System.out.println(list); // [3, 1, 5, 4, 2];  // 변경없음!
System.out.println(sortedList); // [1, 2, 3, 4, 5];
```

2. 스트림은 Iterator처럼 일회용이다. (필요하면 다시 스트림을 생성해야 함)

```
strStream.forEach(System.out::println); // 모든 요소를 화면에 출력(최종연산), forEach 가 최종연산(요소를 소모)
int numOfStr = strStream.count(); // 에러. 스트림이 이미 닫혔음. count()가 최종연산. 중간연산도 할 수 없음
```

3. 최종 연산 전까지 중간연산이 수행되지 않는다. - 지연된 연산
  - 로또 번호 출력하기

    ```
    IntStream intStream = new Random().ints(1, 46); // 1~45 범위의 무한 스트림(유한 스트림도 존재). 무한스트림이라서 스트림의 갯수가 없다. 즉 난수를 끝도 없이 줌.
    intStream.distinct().limit(6).sorted() // 중간 연산(중복제거, 자르기, 정렬)
              .forEach(i -> System.out.print(i + ",")); // 최종 연산
    ```

  - intStream은 무한스트림인데 중복제거가 불가능하다. 그런데도 위의 코드는 가능하다! 왜냐하면 중간연산(중복제거, 자르기, 정렬)은 순차적으로 이루어지는 것이 아니라 체크만 해두고, 나중에 필요할 때 수행된다. 지연된 연산이 이루어지므로.

4. 스트림은 작업을 내부 반복으로 처리한다.

  - 성능은 비효율적이지만 코드가 간결해진다.

  ```
  for (String str : strList)
    System.out.println(str);
  ```

  - 위의 코드가 아래로 변경

  ```
  stream.forEach(System.out::println); // forEach는 최종 연산이다.
  ```

  - 이유는 forEach의 내부 코드에서 찾을 수 있다.

    - for문을 메서드 안으로.

    ```
    void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action); // 매개변수의 널 체크

        for(T t : src) // 내부 반복(for문을 메서드 안으로 넣음)
            action.accept(T);
    }
    ```

5. 스트림의 작업을 병렬로 처리 - 병렬스트림

  - 병렬스트림을 지원

  - parallel() : 스트림 -> 병렬스트림
  - sequential() : 병렬스트림 -> 스트림, 이게 기본

  ```
  Stream<String> strStream = Stream.of("dd", "aaa", "CC", "cc", "b");
  int sum = strStream.parallel() // 병렬 스트림으로 전환(속성만 변경)
                  .mapToInt(s -> s.length()).sum(); // 모든 문자열의 길이의 합
  ```

6. 기본형 스트림 - IntStream, LongStream, DoubleStream

  - 중요한 것은 아님
  - 오토박싱 & 언박싱의 비효율이 제거됨(Stream`<Integer>`대신 IntStream 사용)

    - Stream`<T>` 이므로 기본형이 아니라 T부분에 참조형이 들어가야 한다. 그래서 만약 {1, 2, 3}을 스트림으로 변환하게 되면 오토박싱이 되어 new Integer(1) 이런식으로 들어가게 되며, 반대로 실제로 연산이 이루어 질 때는 언박싱이 이루어지게 된다. 그런데 기본형 스트림을 사용하게 되면 오토박싱 & 언박싱 과정이 이루어지지 않는다.

  - 숫자와 관련된 유용한 메서드를 Stream`<T>` 보다 더 많이 제공
    - Stream`<T>` 같은 경우에는 숫자외에도 여러 타입의 스트림이 가능해야 하므로 count()정도의 메서드 밖에 제공하지 않는데 기본스트림 같은 경우에는 애초부터 숫자이기 때문에 count(), sum(), average() 등의 함수가 다 제공된다.

## 스트림 만들기 - 컬렉션

- 스트림을 만드는 3단계에서 1단계에 대한 내용

  1. `스트림 생성`
  2. 중간 연산(0~n번)
  3. 최종 연산(0~1번)

- Collection 인터페이스의 stream()으로 컬렉션을 스트림으로 변환

  - collcetion 인터페이스의 자손인 List, Set 2개를 구현한 클래스가 stream() 메서드를 가지고 있음

  ```
  Stream<E> stream() // Collection 인터페이스의 메서드
  ```

  ```
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
  Stream<Integer> intStream = list.stream(); // list를 데이터 소스로 하는 새로운 스트림 생성

  // 스트림의 모든 요소를 출력
  intStream.forEach(System.out::print); // 12345, forEach() 최종연산
  intStream.forEach(System.out::print); // 에러. 스트림이 이미 닫혔다.
  ```

  - 에러 없이 다시 생성해서 생성하려면

  ```
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
  Stream<Integer> intStream = list.stream(); // list를 데이터 소스로 하는 새로운 스트림 생성
  intStream.forEach(System.out::print); //  12345, forEach() 최종연산

  intStream = list.stream(); // 재생성 (stream은 1회용) - stream에 대해 최종연산을 수행하면 stream이 닫힌다.
  intStream.forEach(System.out::print); //  12345, forEach() 최종연산
  ```

## 스트림 만들기 - 배열

### 객체 배열로부터 스트림 생성하기

```
Stream<T> Stream.of(T... values) // 가변인자
Stream<T> Stream.of(T[])
Stream<T> Arrays.stream(T[])
Stream<T> Arrays.stream(T[] array, int startInclusive, int endExclusive) // 마지막 index는 포함안됨
```

- 예시

```
Stream<String> strStream = Stream.of("a", "b" ,"c"); // 가변인자
Stream<String> strStream = Stream.of(new String[]{"a", "b", "c"});
Stream<String> strStream = Arrays.stream(new String[]{"a", "b", "c"});
Stream<String> strStream = Arrays.stream(new String[]{"a", "b", "c"}, 0, 3);
```

### 기본형 배열로부터 스트림 생성하기

- 배열에 기본형을 담으면 기본형 스트림이 생성된다.

```
IntStream IntStream.of(int... values) // Stream이 아니라 IntStream
IntStream IntStream.of(int[])
IntStream Arrays.stream(int[])
IntStream Arrays.stream(int[] array, int startInclusive, int endExclusive)
```

- 예시

```
// 기본형 스트림 생성하기
int[] intArr = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(intArr);
// intStream.forEach(System.out::println); // 가능
// System.out.println("count = " + intStream.count()); // 가능. 최종연산
// System.out.println("sum = " + intStream.sum()); // 가능. 최종연산
System.out.println("average = " + intStream.average()); // 가능. 최종연산


// 위를 아래의 Stream으로도 할 수 있다.

Integer[] intArr = {1, 2, 3, 4, 5}; // 자동으로 AutoBoxing이 됨
Stream<Integer> intStream = Arrays.stream(intArr);
// intStream.forEach(System.out::println); // 가능
Systme.out.println("count = " + intStream.count()); // count = 5 나온다 . 그런데 sum(), average() 같은 함수가 없다.
```

### 스트림 만들기 - 임의의 수

- 난수를 요소로 갖는 스트림 생성하기

```
IntStreamintStream = new Random().ints(); // 무한 스트림
intStream.limit(5).forEach(System.out::println); // 5개의 요소만 출력한다.

IntStream intStream = new Random().ints(5); // 크기가 5인 난수 스트림을 반환
```

- Random() 함수의 범위

```
Integer.MIN_VALUE <= ints() <= Integer.MAX_VALUE
LONG.MIN_VALUE <= longs() <= LONG.MAX_VALUE
0.0 <= doubles() <= 1.0

```

- 지정된 범위의 난수를 요소로 갖는 스트림을 생성하는 메서드(Random 클래스)

```
// 무한 스트림
IntStream ints(int begin, int end)
LongStream longs(long begin, long end)
DoubleStream doubles(double begin, double end)

// 유한 스트림
IntStream ints(long streamSize, int begin, int end)
LongStream longs(long streamSize, long begin, long end)
DoubleStream doubles(long streamSize, double begin, double end)
```

- 실습

```
IntStream intStream = new Random().ints(); // 무한스트림
intStream
    .limit(5) // 5개만 자르기. 이게 없으면 무한으로 계속 반환함.
    .forEach(System.out::println);
```

```

IntStream intStream1 = new Random().ints(5); // 유한스트림
intStream1
        .forEach(System.out::println); // 5개 아무거나 출력

IntStream intStream2 = new Random().ints(5, 10); // 유한스트림, 범위 줘서
intStream2
        .limit(10)
        .forEach(System.out::println); // 5 ~ 10 사이에 있는 수 아무거나 10개 반환


IntStream intStream3 = new Random().ints(10, 5, 10); // 유한스트림, 범위 줘서
intStream3
        .forEach(System.out::println); // 5 ~ 10 사이에 있는 수 아무거나 10개 반환
```

### 특정 범위의 정수를 요소로 갖는 스트림 생성하기(IntStream, LongStream)

```
IntStream IntStream.range(int begin, int end) // end 제외
IntStream IntStream.rangeClosed(int begin, int end) // end 포함
```

```
IntStream intStream = IntStream.range(1, 5); // 1, 2, 3, 4
IntStream intStream = IntStream.rangeClsoed(1, 5); // 1, 2, 3, 4, 5
```

### 스트림 만들기 - 람다식 iterate(), generate()

- 람다식을 소스로 하는 스트림 생성하기
  - 무한스트림임, 그래서 limit() 같은 것으로 짤라야 한다.

```
static <T> Stream<T> iterate(T seed, UnaryOperator<T> f) // 이전 요소에 종속적, UnaryOperator는 단항 연산자. 즉, 하나를 넣으면 결과 하나 나옴
static <T> Stream<T> generate(Supplier<T> s) // 이전 요소에 독립적, Supplier는 주기만 하는 것. 즉, 입력은 없고 출력은 있다.
```

- `iterate()`는 이전 요소를 seed로 해서 다음 요소를 계산한다.(종속적)

  ```
  Stream<Integer> evenStream = Stream.iterate(0, n -> n+2); // 0, 2, 4, 6, ...
  ```

  - 즉.

  ```
  n -> n + 2
  ----------
  0 -> 0 + 2
  2 -> 2 + 2
  4 -> 4 + 2
  ...
  ```

  - iterate() 실습

  ```
  Stream<Integer> intStream = Stream.iterate(0, n -> n + 2);
  intStream.forEach(System.out::println); 무한으로 +2 더한값들이 생성
  ```

  ```
  Stream<Integer> intStream = Stream.iterate(0, n -> n + 2);
  intStream.limit(10).forEach(System.out::println); // 10개만 출력 0, 2, 4, 6, ...
  ```

  ```
  Stream<Integer> intStream = Stream.iterate(1, n -> n + 2);
  intStream.limit(10).forEach(System.out::println); // 1부터 홀수 10개 1, 3, 5, 7, ...
  ```

- `generate()`는 seed를 사용하지 않는다.(독립적)

  ```
  Stream<Double> randomStream = Stream.generate(Math::random); // () -> Math.random(); 즉, 메서드 호출한 값을 계속 생성해 나가는 무한 스트림
  Stream<Integer> oneStream = Stream.generate(() -> 1); // 1, 1, 1, 1... 입력은 없고 계속 결과 1만 반환하는 무한 스트림
  ```

  - generate() 실습

  ```
  Stream<Integer> oneStream = Stream.generate(() -> 1);
  oneStream
      .limit(10) // limit(10) 이 없다면 1 무한으로 찍힘
      .forEach(System.out::println); // 1, 1, 1 .. 총 10개
  ```

### 스트림 만들기 - 파일과 빈 스트림

- 파일을 소스로 하는 스트림 생성하기

```
Stream<Path> Files.list(Path dir) // Path는 파일 또는 디렉토리
```

```
Stream<String> Files.lines(Path path) // 파일 내용을 라인 단위로. log파일 분석 같은 경우 유리
Stream<String> Files.lines(Path path, Charset cs)
Stream<String> lines() // BufferedReader클래스의 메서드
```

- 비어있는 스트림 생성하기

```
Stream emptyStream = Stream.empty(); // empty()는 빈 스트림을 생성해서 반환한다.
long count = emptyStream.count(); // count의 값은 0
```

## 스트림의 연산 (개요만)

### 스트림이 제공하는 기능 - 중간 연산과 최종 연산 (개요만)

- 중간 연산 - 연산결과가 `스트림`인 연산. 반복적으로 적용가능
- 최종 연산 - 연산결과가 스트림이 아닌 연산. 단 한번만 적용가능(스트림의 요소를 소모)

```
stream.distinct().limit(5).sorted().forEach(System.out::println)
```

  - distinct()
    - 중간연산(중복제거)
    - 결과가 스트림
  - limit(5)
    - 중간연산(5개 자르기)
    - 결과가 스트림
  - sorted()
    - 중간연산(정렬)
    - 결과가 스트림
  - forEach(System.out::println)
    - 최종연산(출력)
    - (println) 결과가 void
    - 스트림이 닫힘(closed)

  ```
  String[] strArr = {"dd", "aaa", "CC", "cc", "b"};

  Stream<String> stream = Stream.of(strArr); // 문자열 배열이 소스인 스트림
  Stream<String> filteredStream = stream.filter(); // 걸러내기 (중간 연산)
  Stream<String> distinctedStream = stream.distinct(); // 중복제거 (중간 연산)
  Stream<String> sortedStream = stream.sort(); // 정렬 (중간 연산)
  Stream<String> limitedStream = stream.limit(5); // 스트림 자르기 (중간 연산)
  int total = stream.count(); // 요소 개수 세기(최종연산)
  ```

### 스트림의 연산 - 중간 연산 (개요 알아보기)

![image](https://user-images.githubusercontent.com/57219160/136488455-4d84517d-7156-4bb3-bcce-c583e0c0e34c.png)

- limit(5)
  - skip이후에 5개 짜름
- skip(3)
  - 처음 3개 건너뜀
- peek
  - forEach와 비슷.
  - 작업 사이에 넣어서 작업 중간결과를 보고싶을 때 사용
- sorted()
  - 스트림 요소의 기본정렬을 따라서 정렬
- sorted(Comparator`<T>` comparator)
  - 정렬기준을 담아서.
- map, flatMap
  - 스트림 중간연산의 핵심임

### 스트림의 연산 - 최종 연산 (개요 알아보기)

![image](https://user-images.githubusercontent.com/57219160/136488889-5b853ffe-34b3-4d97-b13b-c2ce18517edf.png)

- void forEachOrderd(Consumer<? super T> action)
  - 순서유지
  - `병렬스트림으로 사용할 때`만 작업순서를 유지하고 싶을 때 사용
- Optional`<T>` findAny(), Optional`<T>` findFirst()
  - filter(조건)와 같이 사용한다. 조건에 맞는 요소 중 아무거나 반환(findAny()) 하거나 요소의 첫 번째 것 반환(findFirst())
  - findAny()는 병렬처리할 때 사용
  - findFirst()는 직렬처리할 때 사용
- reduce()와 collect()
  - 스트림 최종연산의 핵심
  - reduce()가 더 중요

## 스트림의 중간연산 1

- skip()
  - 건너뛰기
- limit()
  - 잘라내기
- distinct()
  - 중복제거
- filter()
  - 걸러내기
- sorted()
  - 정렬

### 스트림 자르기 - skip(), limit()

```
Stream<T> skip(long n)  // 앞에서부터 n개 건너뛰기
Stream<T> limit(long maxSize) // maxSize 이후의 요소는 잘라냄
```

```
IntStream intStream = IntStream.rangeClosed(1, 10); // 1 2 3 4 5 6 7 8 9 10
intStream.skip(3).limit(5).forEach(System.out::print); // 4 5 6 7 8, skip(3) 으로 1 2 3 건너뛰고 그 다음 45678 5개 선택된 것
```

### 스트림의 요소 걸러내기 - filter(), distinct()

```
Stream<T> filter(Predicate<? supter T> predicate) // 조건에 맞지 않는 요소 제거
Stream<T> distinct()
```

```
IntStream intStream = IntStream.of(1, 2, 2, 3, 3, 3, 4, 5, 5, 6);
intStream.distinct().forEach(System.out::print); // 1 2 3 4 5 6
```

```
IntStream intStream = IntStream.rangeClosed(1, 10); // 1 2 3 4 5 6 7 8 9 10
intStream.filter(i -> i%2 == 0).forEach(System.out::println); // 2의 배수만 출력 2 4 6 8 10
```

```
IntStream intStream = IntStream.rangeClosed(1, 10); // 1 2 3 4 5 6 7 8 9 10
intStream.filter(i -> i%2 != 0 && i % 3 != 0).forEach(System.out::println); // 1 5 7
intStream.filter(i -> i%2 != 0).filter(i -> i%3 != 0).forEach(System.out::println); // 1 5 7, filter()는 중간연산이고 stream을 반환하므로 여러번 사용해도 된다. 홀수 중에서 3의 배수가 아닌 것들
```

### 스트림 정렬하기 - sorted()

```
Stream<T> sorted() // 스트림 요소의 기본 정렬(Comparable)로 정렬, 특정한 기준을 주지 않을 시.
Stream<T> sorted(Comparator<? super T> comparator) // 지정된 Comparator로 정렬, 특정한 기준을 준 것.
```

![image](https://user-images.githubusercontent.com/57219160/136490990-d452ebcf-32f1-4cad-8303-100a9dda9661.png)

- strStream.sorted(Comparator.reverOrder())와 strStream.sorted(Comparator.`<String>`naturalOrder().reversed()) 와 결과는 같음. naturalOrder() 정상순의 다시 reverse() 역순이므로.
- strStream.sorted(String.CASE_INSENSITIVE_ORDER)
  - String 클래스에는 `static Comparator<String> CASE_INSENSITIVE_ORDER = new CaseInsensitiveComparator();` 가 있다. CASE_INSENSITIVE_ORDER가 Comparator인데 이 말은 String클래스가 Comparator를 이미 만들어서 갖고 있다는 것. 자주 사용하니까 이미 만들어 둔 것임.
- strStream.sorted(Comparator.comparing(String::length))
  - bddCCccaaa
  - b가 1개, d가 2개, C가 2개, c가 2개, a가 3개 => 1 2 2 2 3 순으로 정렬.

### Comparator의 comparing()으로 정렬 기준을 제공(정렬 기준 1개일 때)

- comparing의 반환타입은 Comparator

```
comparing(Function<T, U> keyExtractor)
comparing(Function<T, U> keyExtractor, Comparator<U> keyComparator)
```

- 아래의 sorted()는 원래 `Stream<T> sorted(Comparator comparator)` 므로 파라미터로 Comparator를 받으니까, comparing() 의 반환타입도 Comparator니까 맞음.

```
studentStream.sorted(Comparator.comparing(Student::getBan)) // 반별로 정렬
            .forEach(System.out::println);
```

### 추가 정렬 기준을 제공할 때는 thenComparing()을 사용 (정렬 기준 여러개일 때)

```
thenComparing(Comparator<T> other)
thenComparing(Function<T, U> keyExtractor)
thenComparing(Function<T, U> keyExtractor, Compartor<U> keyComp)
```

- 아래서 정렬기준 3개. 더 붙여서 추가해도 됨

```
studentStream.sorted(Comparator.comparing(Student::getBan) // 반별로 정렬
                  .thenComparing(Student::getTotalScore) // 총점별로 정렬
                  .thenComparing(Student::getName)) // 이름별로 정렬
                  .forEach(System.out::println);)
```

### 실습

```
class Student implements Comparable<Student> {
    String name;
    int ban;
    int totalScore;

    Student(String name, int ban, int totalScore) {
        this.name = name;
        this.ban = ban;
        this.totalScore = totalScore;
    }

    public String toString() {
        return String.format("[%s, %d, %d]", name, ban, totalScore);
    }

    String getName() { return name;}
    int getBan() { return ban;}
    int getTotalScore() { return totalScore;}

    // 총점 내림차순을 기본 정렬로 한다. 300 200 100..
    @Override
    public int compareTo(Student s) {
        return s.totalScore - this.totalScore;
    }
}


public class Test {
    public static void main(String[] args) {

        Stream<Student> studentStream = Stream.of(
                    new Student("이자바", 3, 300),
                    new Student("김자바", 1, 200),
                    new Student("안자바", 2, 100),
                    new Student("박자바", 2, 150),
                    new Student("소자바", 1, 200),
                    new Student("나자바", 3, 290),
                    new Student("김자바", 3, 180)
        );

        studentStream.sorted(Comparator.comparing(Student::getBan) // 1. 반별 정렬
                .thenComparing(Comparator.naturalOrder())) // 2. 기본정렬
                .forEach(System.out::println);
    }
}

```

- 결과는

```
[김자바, 1, 200]
[소자바, 1, 200]
[박자바, 2, 150]
[안자바, 2, 100]
[이자바, 3, 300]
[나자바, 3, 290]
[김자바, 3, 180]
```

- 만약 역순으로 하고 싶다면 각각 뒤에 reverse() 붙이면 됨

```
studentStream.sorted(Comparator.comparing(Student::getBan).reversed() // 1. 반별 정렬
        .thenComparing(Comparator.naturalOrder()).reversed()) // 2. 기본정렬
        .forEach(System.out::println);
```

## 스트림의 중간연산 2

- map()
  - 변환
- peek()
  - forEach()와 비슷하지만 peek()는 중간연산, forEach()는 최종연산
- flatmap()
  - 변환, 스트림의 스트림 -> 스트림으로 변환

### 스트림의 요소 변환하기 - map()

```
Stream<R> map(Function<? super T, ? extends R> mapper) // Stream<T> -> Stream<R>
```

```
Stream<File> fileStream = Stream.of(new File("Ex1.java"), new File("Ex1")),
            new File("Ex1.bak"), new File("Ex2.java"), new File("Ex1.txt"));

Stream<String> filenameStream = fileStream.map(File::getName); // map()을 이용해서 결국 File -> String 스트림으로 변환하는 것.
filenameStream.forEach(System.out::println); // 스트림의 모든 파일의 이름을 출력
```

![image](https://user-images.githubusercontent.com/57219160/136503821-d84a575d-5eb9-4f89-ab51-0f0e3f3e6a85.png)

- ex) 파일 스트림(Stream`<File>`)에서 파일 확장자(대문자)를 중복없이 뽑아내기

```
fileStream.map(File::getName)                   // Stream<File> -> Stream<String>
  .filter(s->s.indexOf('.')!=-1)                // 확장자가 없는 것은 제외
  .map(s->s.substring(s.indexOf('.')+1))        // Stream<String> -> Stream<String>, 확장자만 뽑아낸 것. Ex1.bak -> bak
  .map(String::toUpperCase)                     // Stream<String> -> Stream<String>, 대문자로 변경
  .distinct()                                   // 중복 제거
  .forEach(System.out::println); // JAVABAKTXT
```

- 실습

```
public class Test2 {
    public static void main(String[] args) {
        File[] fileArr = { new File("Ex1.java"), new File("Ex1.bak"),
                new File("Ex2.java"), new File("Ex1"), new File("Ex1.txt")
        };

        Stream<File> fileStream = Stream.of(fileArr);

        // map()으로 Stream<File>을 Stream<String>으로 변환
        Stream<String> filenameStream = fileStream.map(File::getName);
        filenameStream.forEach(System.out::println); // 모든 파일의 이름을 출력(최종연산이라 다 소모함)

        fileStream = Stream.of(fileArr); // 스트림을 다시 생성

        fileStream.map(File::getName)               // Stream<File> -> Stream<String>
                .filter(s -> s.indexOf('.')!=-1)    // 확장자가 없는 것은 제외
                .map(s -> s.substring(s.indexOf('.') + 1)) // 확장자만 추출
                .map(String::toUpperCase)                   // 모두 대문자로 변환
                .distinct()                         // 중복 제거
                .forEach(System.out::println);        // JAVABAKTXT

        System.out.println();
    }
}
```

### 스트림의 요소를 소비하지 않고 엿보기 - peek()

- 작업 중간에 잘 진행되고 있는지 확인시 사용. `디버깅 용도`

```
Stream<T> peek(Consumer<? super T> action) // 중간 연산(스트림의 요소를 소비x)
void      forEach(Consumer<? super T> action) // 최종 연산(스트림의 요소를 소비O)
```

- 아래처럼 여러작업을 수행할 때 중간중간에 잘 진행되고 있는지 확인할 때 사용한다. 

```
fileStream.map(File::getName)     // Stream<File> -> Stream<String>
      .filter(s -> s.indexOf('.')!=-1) // 확장자가 없는 것은 제외
      .peek(s -> System.out.printf("filename=%s%n", s)) // 파일명을 출력한다.(중간작업결과 확인)
      .map(s -> s.substring(s.indexOf('.')+1)) // 확장자만 추출
      .peek(s -> System.out.printf("extension=%s%n", s)) // 확장자를 출력한다.(중간작업결과 확인)
      .forEach(System.out::println); // 최종연산 스트림을 소비
```

- 실습

```
public class Test {
    public static void main(String[] args) {
        File[] fileArr = { new File("Ex1.java"), new File("Ex1.bak"),
                new File("Ex2.java"), new File("Ex1"), new File("Ex1.txt")
        };

        Stream<File> fileStream = Stream.of(fileArr);

        // map()으로 Stream<File>을 Stream<String>으로 변환
        Stream<String> filenameStream = fileStream.map(File::getName);
        filenameStream.forEach(System.out::println); // 모든 파일의 이름을 출력(최종연산이라 다 소모함)

        fileStream = Stream.of(fileArr); // 스트림을 다시 생성

        fileStream.map(File::getName)               // Stream<File> -> Stream<String>
                .filter(s -> s.indexOf('.')!=-1)    // 확장자가 없는 것은 제외
                .peek(s -> System.out.printf("filename=%s%n", s))  // 중간 확인
                .map(s -> s.substring(s.indexOf('.') + 1)) // 확장자만 추출
                .peek(s -> System.out.printf("extension=%s%n", s))  // 중간 확인
                .map(String::toUpperCase)                   // 모두 대문자로 변환
                .distinct()                         // 중복 제거
                .forEach(System.out::println);        // JAVABAKTXT

        System.out.println();
    }
}

```

### 스트림의 스트림을 스트림으로 변환 - flatMap()

- 스트림의 하나하나가 스트링 배열

```
Stream<String[]> strArrStrm = Stream.of(new String[]{"abc", "def", "ghi"},
                                        new String[]{"ABC", "GHI", "JKLMN"});
```

- 스트림의 스트림

```
Stream<Stream<String>> strStrStrm = strArrStrm.map(Arrays::stream);
```

![image](https://user-images.githubusercontent.com/57219160/136509742-fc531b68-f13c-4675-af25-ec3af609abc5.png)

- 하지만 원래 내가 원했던 것은, 문자의 배열들을 하나로 합치고 싶었던 것(자주 발생함). 위처럼 각각을 하나의 스트림으로 따로 만들지 말고.

```
Stream<String> strStrStrm = strArrStrm.flatMap(Arrays::stream); // Arrays.stream(T[])
```

![image](https://user-images.githubusercontent.com/57219160/136510021-ddfd8fae-bffe-45b5-ac1e-cd9702d82c98.png)

- 실습

  ```
  public class Test {
      public static void main(String[] args) {
          Stream<String[]> strArrStrm = Stream.of(
              new String[]{"abc", "def", "jkl"},
              new String[]{"ABC", "GHI", "JKL"}
          );

  //        Stream<Stream<String>> strStrmStrm = strArrStrm.map(Arrays::stream); // 이렇게 하면 Stream 안에 Stream이 존재하게 되어 우리가 원하는 결과가 나오지 않는다.
          Stream<String> strStrm = strArrStrm.flatMap(Arrays::stream);

          strStrm.map(String::toLowerCase)    // 스트림의 요소를 모두 소문자로 변경
                  .distinct() // 중복제거
                  .sorted()   // 정렬
                  .forEach(System.out::println);
          System.out.println();

          String[] lineArr = {
                  "Believe or not It is true",
                  "Do or do not There is no try"
          };

          Stream<String> lineStream = Arrays.stream(lineArr); // 두 문장에 있는 단어 하나하나(String)를 Stream의 요소로 만들고 싶은 것
          lineStream.flatMap(line -> Stream.of(line.split(" +")))
                  .map(String::toLowerCase)
                  .distinct()
                  .sorted()
                  .forEach(System.out::println);
          System.out.println();
      }
  }
  ```

  - `lineStream.flatMap(line -> Stream.of(line.split(" +")))`
    - line을 split하면 String -> String[]로 바꾸는 것. " +"는 정규식(Regular Expression)인데 하나이상의 공백. 이라는 뜻. cf) " " 는 공백하나
    - Stream`<String>` ---map()을 사용하면--> Stream`<Stream<String>>` 이 되버린다.
    - Stream`<String>` ---flatMap()을 사용하면--> Stream`<String>` 이 된다.
    - 그래서 공백을 제외한 Believe, or, not 등의 단일 단어들로 이루어진 Stream`<String>`이 나온다.

## Optional<T>

- [Here](https://highjune.dev/java/optional/)

## 스트림의 최종연산

1. forEach()
2. allMatch()
3. anyMatch()
4. noneMatch()
5. reduce() - 핵심
6. findFirst()
7. findAny()
8. collect()

### 스트림의 모든 요소에 지정된 작업을 수행 - forEach(), forEachOrdered()

```
void forEach()(Consumer<? super T> action) // 병렬스트림인 경우 순서가 보장되지 않음, 꼭 순서를 유지하지 않아도 되는 경우에는 forEachOrdered 보다 더 빠름
void forEachOrdered(Consumer<? super T> action) // 병렬스트림인 경우에도 순서가 보장됨
```

- sequential()
  - 직렬 스트림
  - 스트림의 작업을 직렬로 처리하라는 뜻
  - 하나의 쓰레드가 순서대로 처리

```
IntStream.range(1, 10).sequential().forEach(System.out::print); // 123456789
IntStream.range(1, 10).sequential().forEachOrdered(System.out::print) // 123456789
```

- parellel()
  - 병렬 스트림
  - 여러 쓰레드가 나눠서 작업하는 것
  - 데이터가 많을때는 병렬로 처리할 수도 있다.

```
IntStream.range(1, 10).parellel().forEach(System.out::print); // 523857172, 섞여서 나옴. 여러 쓰레드가 나눠서 처리하므로 순서보장x
IntStream.range(1, 10).parellel().forEachOrdered(System.out::print) // 123456789, 병렬처리함에도 순서보장이 됨
```

### 조건 검사 - allMatch(), anyMatch(), noneMatch()

```
boolean allMatch (Predicate<? super T> predicate) // 모든 요소가 조건을 만족시키면 true
boolean anyMatch (Predicate<? super T> predicate) // 한 요소라도 조건을 만족시키면 true
boolean noneMatch (Predicate<? super T> predicate) // 모든 요소가 조건을 만족시키지 않으면 true
```

- 100점 이하의 낙제자가 있는지?

```
Stream<Student> stuStream = new ~~(생략)~~
boolean hasFailedStu = stuStream.anyMatch(s -> s.getTotalScore() <= 100); // 한 명이라도 있으면 true
```

### 조건에 일치하는 요소 찾기 - findFirst(), findAny()

- 조건에 맞는 것이 없을 수도 있으므로, 즉 결과가 null을 반환할 수도 있으므로 Optional`<T>`로 반환

```
Optional<T> findFirst() // 첫 번째 요소를 반환. 순차 스트림에 사용. 찾다가 처음으로(first) 조건에 맞는 것을 반환.
Optional<T> findAny() // 아무거나 하나를 반환. 병렬 스트림에 사용. 여러 쓰레드들 중에서 조건을 먼저 발견한 쓰레드가 결과를 반환. 조건에 맞는 것들이 여러개 있다고 해도 각각에 해당하는 쓰레드들 중에서 무엇이 반환할지 모르니까 any.
```

```
Stream<Student> stuStream = new ~~(생략)~~
Optional<Student> result = stuStream.filter(s -> s.getTotalScore() <= 100).findFirst(); // 낙제자 중 첫번째 요소 반환
Optional<Student> result = parallelStream.filter(s -> s.getTotalScore() <= 100).findAny(); // 병렬스트림일 경우 여러 요소중에 먼저 발견한 쓰레드가 반환하는 요소 반환
```

### 스트림의 요소를 하나씩 줄여가며 누적연산(accumulator) 수행 - reduce()

- 매우 중요
- 스트림의 최종 연산은 모두 reduce()로 만들어져 있다. ex) count(), max(), min(), collect() 등

  ```
  Optional<T> reduce(BinaryOperator<T> accumulator)
  T           reduce(T identity, BinaryOperator<T> accumulator)
  U           reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)
  ```

- identity
  - 초기값
  - 핵심
- accumulator
  - 이전 연산결과와 스트림의 요소에 수행할 연산
  - 핵심
- combiner
  - 병렬처리된 결과를 합치는데 사용할 연산(병렬 스트림)
  - 크게 중요치 않음

- `Optional<T> reduce(BinaryOperator<T> accumulator)`

  - 초기값(identity)이 없고, 스트림의 요소가 하나도 없을 때 null을 반환할 수도 있다. 그래서 반환값이 Optional`<T>`

- `T reduce(T identity, BinaryOperator<T> accumulator)`

  - 초기값(identity)가 있고, 스트림의 요소가 하나도 없을 때 identity를 반환하게 된다. 반환값이 T

- count, sum

```
// int reduce(int identity, IntBinaryOperator op)
int count = intStream.reduce(0, (a, b) -> a + 1);
int sum = intStream.reduce(0, (a, b) -> a + b);
```

- 위의 작업은 아래와 같다.(reduce의 핵심 연산. 중요)

```
int a = identity; // 초기값이자 누적결과를 저장할 변수
for (int b : stream)
    a = a + b; // sum(), 이곳에 연산식이 들어간다. min()이든, count()든, 새로운 식이든
```

- max, min

```
int max = intStream.reduce(Integer.MIN_VALUE, (a, b) -> a > b ? a : b); // max()
int min = intStream.reduce(Integer.MAX_VALUE, (a, b) -> a < b ? a : b); // min()
```

### 실습

```
public class Test {
    public static void main(String[] args) {
        String[] strArr = {
                "Inheritance", "Java", "Lambda", "stream",
                "OptionalDouble", "IntStream", "count", "sum"
        };

        Stream.of(strArr)
                .parallel() // 병렬로 처리하면 출력 순서 보장X. 실행시마다 다름
//                .forEach(System.out::println);
                .forEachOrdered(System.out::println); // 병렬로 처리하면서 순서도 유지하고자 한다면

        boolean noEmptyStr = Stream.of(strArr).noneMatch(s -> s.length() == 0);
        Optional<String> sWord = Stream.of(strArr)
                                    .parallel() // 랜덤으로 처리
                                    .filter(s -> s.charAt(0) == 's')
//                                    .findFirst(); // Stream 나옴
                                    .findAny();

        System.out.println("noEmptyStr = " + noEmptyStr); // noEmptyStr = true
        System.out.println("sWord = " + sWord.get()); // sWord = stream (parallel() 때문에 sum, stream 중 랜덤으로 나옴, 병렬로 처리하니깐 어떤 것이 발견될지 모름)

        // Stream<String>을 Stream<Integer>으로 변환, 각 요소를 객체로 다룸. (s) -> s.length()
        Stream<Integer> intStream0 = Stream.of(strArr).map(String::length);  // IntStream {11, 4, 6, 6, 14, 9, 5, 3} 생성

        // Stream<String >을 IntStream으로 변환. IntStream기본형 스트림. (성능 때문에). 각 요소를 기본형으로 다룸
        IntStream intStream1 = Stream.of(strArr).mapToInt(String::length);
        IntStream intStream2 = Stream.of(strArr).mapToInt(String::length);
        IntStream intStream3 = Stream.of(strArr).mapToInt(String::length);
        IntStream intStream4 = Stream.of(strArr).mapToInt(String::length);

        int count = intStream1.reduce(0, (a, b) -> a + 1); // count = 8, 요소 수
        int sum = intStream2.reduce(0, (a, b) -> a + b); // sum = 58, 단어수 합

//        OptionalInt max = intStream3.reduce(Integer::max); // 초기값이 없어 산출값이 없을수도 있으므로 Optional로 반환
        OptionalInt max = IntStream.empty().reduce(Integer::max); // 초기값이 없어 산출값이 없을수도 있으므로 Optional로 반환
        OptionalInt min = intStream3.reduce(Integer::min);

        System.out.println("count = " + count);
        System.out.println("sum = " + sum);
//        System.out.println("min = " + max.getAsInt()); // 안에 요소 없으면 에러 반환
        System.out.println("max = " + max.orElse(0)); //  그래서 요소가 없으면 0을 반환하는 것으로 수정
        System.out.println("min = " + min.getAsInt()); // 안에 요소가 없으면 에러 반환


    }
}

```

## collect() 와 Collector

### collect()는 Collector 를 매개변수로 하는 스트림의 최종연산

- collect() : 최종연산
- Collector는 인터페이스
- Collectors : 클래스(Collector는 구현)

```
Object collect(Collector collector) // Collector를 구현한 클래스의 객체를 매개변수로
object colelct(Supplier supplier, BiConsumer accumulator, BiConsumer combiner) // 잘 안 쓰임
```

- reduce() 와 collect() 의 차이는?

  - reduce() 전체 리듀싱
  - collect() 그룹별 리듀싱, 전체도 가능
  - 사실 둘은 거의 같은 것

### Collector는 수집(collect)에 필요한 메서드를 정의해 놓은 인터페이스

![image](https://user-images.githubusercontent.com/57219160/136884247-e9a0e75b-2d8f-4e55-aa09-46ae58ccdad5.png)

- combiner() 는 쓰레드가 각각 병렬작업했을 때 다 합치는 것
- supplier() 와 accumulator()가 핵심
  - StringBuilder::new -> 초기화
  - (sb, s) -> sb.append(s) -> 누적
  - 즉, reduce(identity, accumulator)에서 identity-초기화, accumulator-누적수행작업 과 동일
  - combiner()는 병렬작업일 때 합치는 것이고, finisher()는 변환이 필요하면 쓰는 것.
  - 그래서 위의 메서드들을 사용하려면 Collector인터페이스를 구현해야 한다. 하지만 아래의 Collectors 클래스가 이미 다 구현해놨다.
  - 직접 구현할 일이 거의 없다.
  - Collectors 클래스 사용만 잘하면 된다.

### Collectors클래스는 다양한 기능의 컬렉터(Collector를 구현한 클래스) 를 제공

![image](https://user-images.githubusercontent.com/57219160/136885145-2b995712-51f5-4e3e-86bb-f1503db2f977.png)

### 스트림을 컬렉션으로 변환 - toList(), toSet(), toMap(), toCollection()

- collect() 메서드를 통해 스트림 -> 컬렉션으로 변환
  ![image](https://user-images.githubusercontent.com/57219160/136885593-7e1a66ba-4459-405a-9500-f5e20f6a1963.png)

### 스트림을 배열로 변환 - toArray()

```
Student[] stuNames = studentStream.toArray(Student[]::new); // OK
Student[] stuNames = studentStream.toArray(); // 에러, 반환타입이 Object임.
Student[] stuNames = (Student[])studentStream.toArray(); // OK. 자동 형변환이 안된다는 뜻
Object[] stuNames = studentStream.toArray(); // OK
```

### 스트림의 통계정보 제공 - counting(), summingInt(), maxBy(), minBy(), ...

- `갯수`
  - stream에서 count()를 통해서 얻을 수 있었듯이, collect 메서드를 통해서도 얻을 수 있다.
    - stream의 count()는 전체만 가능한데, collect로 카운팅을 하면 그룹별로 카운팅할 수 있으므로 유용
    - ex) 남자 xx명, 여자 xx명

```
long count = stuStream.count();
long count = stuStream.collect(counting()); // Collectors.counting(), 즉 static import 한 것.
```

- `합계`
  - stream에서 sum을 할 수 있지만 collect()메서드를 통해서도 마찬가지. 그룹별로 할 수 있어 유용
    - sum()은 전체만 가능한데, collect()는 그룹별 합계가 가능

```
long totalScore = stuStream.mapToInt(Student::getTotalScore).sum(); // IntStream의 sum()
long totalScore = stuStream.collect(summingInt(Student::getTotalScore));
```

- `최대값`
  - max()는 전체요소 중 최대값, collect()는 그룹별로 가능. ex) 남자1등, 여자1등
  - Comparator는 비교기준. 여기서는 총점(getTotalScore)

```
OptionalInt topScore = studentStream.mapToInt(Student::getTotalScore).max();
Optional<Student> topStudent = stuStream
                                .max(Comparator.comparingInt(Student::getTotalScore));
Optional<Studnet> topStudent = stuStream
                              .collect(maxBy(Comparator.comparingInt(Student::getTotalScore)));
```

### 스트림을 리듀싱 - reducing()

- Collectors 클래스가 가지고 있는 메서드
- reduce()와 기능이 같다.
  - reduce()는 전체요소에 대한 리듀싱(sum, count 같은 것)
  - Collectors.reducing()은 그룹별 리듀싱, 전체도 가능

```
Collector reducing(BinaryOperator<T> op)
Collector reducing(T identity, BinaryOperator<T> op) // T identity 는 초기화, BinaryOperator<T> op 은 누적작업을 의미
Collector reducing(U identity, Function<T, U> mapper, BinaryOperator<U> op) // map + reduce. 리듀싱하기 전에 변환(ex. map)이 필요할 경우 사용
```

- `Collector reducing(T identity, BinaryOperator<T> op)` 를 대부분 사용한다.

```
IntStream intStream = new Random().ints(1, 46).distinct().limit(6);

OptionalInt max = intStream.reduce(Integer::max); // 전체 리듀싱
Optional<Integer> max = intStream.boxed().collect(reducing(Integer::max)); // 그룹별 리듀싱 가능
```

```
long sum = intStream.reduce(0, (a, b) -> a + b);
long sum = intStream.boxed().collect(reducing(0, (a, b) -> a + b));
```

```
int grandTotal = stuStream.map(Student::getTotalScore).reduce(0, Integer::sum);
int grandTotal = stuStream.collect(reducing(0, Student::getTotalScore, Integer::sum));
```

### 문자열 스트림의 요소를 모두 연결 - joining()

- joining() 반환타입이 Collector
- joining()은 앞에 Collectors가 가지고 있는 메서드. static import해서 클래스명 생략

```
String studentNames = stuStream.map(Student::getName).collect(joining()); //  Stream<Studnet> -> Stream<String>.
String studentNames = stuStream.map(Student::getName).collect(joining(",")); // 구분자임. 김자바, 이자바, 박자바, ...
String studentNames = stuStream.map(Student::getName).collect(joining(",", "[", "]")); // ,는 구분자, [와 ]는 앞뒤로. 그래서 [김자바, 이자바, 박자바, ...]
String studentInfo = stuStream.collect(joining(",")); // Student의 toString()으로 결합
```

## 스트림의 그룹화와 분할

- 분할을 해서 그룹별로 원하는 값을 얻는다. 그런데 분할을 하는 방법이 크게 2가지. 2분할, n분할

### 1. partitioningBy()는 스트림을 2분할한다.

- Collectors 클래스에 있는 메서드

```
Collector partitioningBy(Predicate predicate)
Colelctor partitioningBy(Predicate predicate, Collector downstream)
```

```
Map<Boolean, List<Student>> stuBySex = stuStream
                              .collect(partitioningBy(Student::isMale)); // 학생들을 성별로 분할, 남/여로 나눠진다 (2분할). 여기서 사실 groupingBy를 사용해도 되지만 굳이. 그리고 2분할이라면 partitioningBy가 더 빠르다.
List<Student> maleStudent = stuBySex.get(true); // Map에서 남학생 목록을 얻는다.
List<Student> maleStudent = stuBySex.get(false); // Map에서 여학생 목록을 얻는다.
```

- 분할(남/녀)한 것을 count
  - counting()도 Collectors.counting()이다. static import됨
  - count된 것이 Long타입으로 나옴

```
Map<Boolean, Long> stuNumBySex = stuStream
                                  .collect(partitioningBy(Student::isMale, counting())); // 분할 + 통게
System.out.println("남학생 수 :" + stuNumBySex.get(true)); // 남학생 수 : 8
System.out.println("여학생 수 :" + stuNumBySex.get(false)); // 여학생 수 : 10
```

- 학생을 성별로 나누고 최대값
  - 최대값을 구하는 기준은 성적. getScore

```
Map<Boolean, Optional<Student>> topScoreBySex = stuStream
        .collect(partitioningBy(Student::isMale, maxBy(comparingInt(Student::getScore))));
System.out.println("남학생 1등 : " + topScoreBySex.get(true)); // 남학생 1등 : Optional[[나바자, 남, 1, 1, 300]]
System.out.println("여학생 1등 : " + topScoreBySex.get(false)); // 여학생 1등 : Optional[[김지미, 여, 1, 1, 250]]
```

```
Map<Boolean, Map<Boolean, List<Student>>> failedStuBySex = stuStream // 다중 분할 (학생을 남/녀로 나누고 각각에서 합/불로 또 나눔)
                                                          .collect(partitioningBy(Student::isMale // 1. 성별로 분할(남/녀)
                                                          , partitioningBy(s -> s.getScore() < 150))); // 2. 성적으로 분할(불합격/합격)
List<Student> failedMaleStu = failedStuBySex.get(true).get(true); // 불합격이 true. 위에서 불합격조건을 주었기 때문에.(150점 밑)
List<Student> failedFemaleStu = failedStuBySex.get(false).get(true);
```

- 실습

```
class Student2 {
    String name;
    boolean isMale; // 성별
    int hak;        // 학년
    int ban;        // 반
    int score;

    Student2(String name, boolean isMale, int hak, int ban, int score) {
        this.name = name;
        this.isMale = isMale;
        this.hak = hak;
        this.ban = ban;
        this.score = score;
    }

    String getName() { return name; }
    boolean isMale() { return isMale; }
    int getHak() { return hak; }
    int getBan() { return ban; }
    int getScore() { return score; }

    public String toString() {
        return String.format("[%s, %s, %d학년 %d반, %3d점]",
                name, isMale ? "남" : "여", hak, ban, score);
    }

    // groupingBy()에서 사용
    enum Level {HIGH, MId, LOW} // 성적을 상, 중, 하 세 단계로 분류

}

class Test {
    public static void main(String[] args) {

        Student2[] stuArr = {
                new Student2("나자바", true, 1, 1, 300),
                new Student2("김지미", false, 1, 1, 250),
                new Student2("김자바", true, 1, 1, 200),
                new Student2("이지미", false, 1, 2, 150),
                new Student2("남자바", true, 1, 2, 100),

                new Student2("안지미", false, 1, 2, 50),
                new Student2("황지미", false, 1, 3, 100),
                new Student2("강지미", false, 1, 3, 150),
                new Student2("이자바", true, 1, 3, 200),
                new Student2("나자바", true, 2, 1, 300),

                new Student2("김지미", false, 2, 1, 250),
                new Student2("김자바", true, 2, 1, 200),
                new Student2("이지미", false, 2, 2, 150),
                new Student2("남자바", true, 2, 2, 100),
                new Student2("안지미", false, 2, 2, 50),

                new Student2("황지미", false, 2, 3, 100),
                new Student2("강지미", false, 2, 3, 150),
                new Student2("이자바", true, 2, 3, 200)
        };

        System.out.printf("1. 단순분할(성별로 분할)%n");
        Map<Boolean, List<Student2>> stuBysex = Stream.of(stuArr)
                      .collect(partitioningBy(Student2::isMale));

        List<Student2> maleStudent = stuBysex.get(true);
        List<Student2> femaleStudent = stuBysex.get(false);

        for(Student2 s : maleStudent) System.out.println(s);
        for(Student2 s : femaleStudent) System.out.println(s);

        System.out.printf("%n2. 단순분할 + 통계(성별 학생수)%n");
        Map<Boolean, Long> stuNumBySex = Stream.of(stuArr)
                .collect(partitioningBy(Student2::isMale, counting()));

        System.out.println("남학생 수 : " + stuNumBySex.get(true));
        System.out.println("여학생 수 : " + stuNumBySex.get(false));

        System.out.printf("%n3. 단순분할 + 통계(성별 1등)%n");
        Map<Boolean, Optional<Student2>> topScoreBySex = Stream.of(stuArr)
                        .collect(partitioningBy(Student2::isMale,
                            maxBy(comparingInt(Student2::getScore))
                        ));
        System.out.println("남학생 1등 : " + topScoreBySex.get(true));
        System.out.println("여학생 1등 : " + topScoreBySex.get(false));

        Map<Boolean, Student2> topScoreBySex2 = Stream.of(stuArr)
            .collect(partitioningBy(Student2::isMale,
                   collectingAndThen(
                           maxBy(comparingInt(Student2::getScore)), Optional::get
                   )
            ));
        System.out.println("남학생 1등 : " + topScoreBySex2.get(true));
        System.out.println("여학생 1등 : " + topScoreBySex2.get(false));

        System.out.printf("%n4. 다중분할(성별 불합격자, 100점 이하)%n");

        Map<Boolean, Map<Boolean, List<Student2>>> failedStuBySex =
                Stream.of(stuArr).collect(partitioningBy(Student2::isMale,  // 성별로 먼저 나누고
                    partitioningBy(s -> s.getScore() <= 100))               // 100점 이하인지 또 나누고
            );
        List<Student2> failedMaleStu = failedStuBySex.get(true).get(true);      // 남자 & 100점 이하
        List<Student2> failedFemaleStu = failedStuBySex.get(false).get(true);   // 여자 & 100점 이하

        for(Student2 s : failedMaleStu) System.out.println(s);
        for(Student2 s : failedFemaleStu) System.out.println(s);
    }
}


```

- 실습 결과

```
1. 단순분할(성별로 분할)
[나자바, 남, 1학년 1반, 300점]
[김자바, 남, 1학년 1반, 200점]
[남자바, 남, 1학년 2반, 100점]
[이자바, 남, 1학년 3반, 200점]
[나자바, 남, 2학년 1반, 300점]
[김자바, 남, 2학년 1반, 200점]
[남자바, 남, 2학년 2반, 100점]
[이자바, 남, 2학년 3반, 200점]
[김지미, 여, 1학년 1반, 250점]
[이지미, 여, 1학년 2반, 150점]
[안지미, 여, 1학년 2반,  50점]
[황지미, 여, 1학년 3반, 100점]
[강지미, 여, 1학년 3반, 150점]
[김지미, 여, 2학년 1반, 250점]
[이지미, 여, 2학년 2반, 150점]
[안지미, 여, 2학년 2반,  50점]
[황지미, 여, 2학년 3반, 100점]
[강지미, 여, 2학년 3반, 150점]

2. 단순분할 + 통계(성별 학생수)
남학생 수 : 8
여학생 수 : 10

3. 단순분할 + 통계(성별 1등)
남학생 1등 : Optional[[나자바, 남, 1학년 1반, 300점]]
여학생 1등 : Optional[[김지미, 여, 1학년 1반, 250점]]
남학생 1등 : [나자바, 남, 1학년 1반, 300점]
여학생 1등 : [김지미, 여, 1학년 1반, 250점]

4. 다중분할(성별 불합격자, 100점 이하)
[남자바, 남, 1학년 2반, 100점]
[남자바, 남, 2학년 2반, 100점]
[안지미, 여, 1학년 2반,  50점]
[황지미, 여, 1학년 3반, 100점]
[안지미, 여, 2학년 2반,  50점]
[황지미, 여, 2학년 3반, 100점]
```

### 2. groupingBy()는 스트림을 n분할한다.

- Collectors 클래스에 있는 메서드

```
Collector groupingBy(Function classifier)
Collector groupingBy(Function classifier, Collector downstream)
Collector groupingBy(Function classifier, Supplier mapFactory, Collector downstream)
```

### 스트림의 요소를 그룹화

- 반으로 나눔

```
Map<Integer, List<Student>> stuByBan = stuStream                // 학생을 반별로 그룹화
              .collect(groupingBy(Student::getBan, toList()));  // toList() 생략가능
```

- 학년으로 나누고 다시 반별로 그룹화

```
Map<Integer, Map<Integer, List<Student>>> stuByHakAndBan = stuStream // 다중 그룹화
                .collect(groupingBy(Student::getHak),                 // 1. 학년별 그룹화
                         groupingBy(Student::getBan                   // 2. 반별 그룹화
                  ));
```

- 학년으로 나누고 반으로 나눈 후 학생들 성적을 map을 사용하여 등급 나눔

```
Map<Integer, Map<Integer, Set<Student.Level>>> stuByHakAndBan = stuStream
    .collect(
      groupingBy(Student::getHak, groupingBy(Student::getBan,     // 다중 그룹화(학년별, 반별)
            mapping(s -> {    // 성적등급(Level)으로 변환. List<Student> -> Set<Student.Level>
              if (s.getScore() >= 200) return Student.Level HIGH,
              else if (s.getScore() >= 100) return Student.Level MID,
              else                          return Student.Level LOW;
            }, toSet()) // mapping()              // enum Level {HIGH, MID, LOW}
      )) // groupingBy()
  ); // collect()
```

- 실습

```
class Student3 {
    String name;
    boolean isMale; // 성별
    int hak;        // 학년
    int ban;        // 반
    int score;

    Student3(String name, boolean isMale, int hak, int ban, int score) {
        this.name = name;
        this.isMale = isMale;
        this.hak = hak;
        this.ban = ban;
        this.score = score;
    }

    String getName() { return name; }
    boolean isMale() { return isMale; }
    int getHak() { return hak; }
    int getBan() { return ban; }
    int getScore() { return score; }

    public String toString() {
        return String.format("[%s, %s, %d학년 %d반, %3d점]",
                name, isMale ? "남" : "여", hak, ban, score);
    }

    // groupingBy()에서 사용
    enum Level {HIGH, MID, LOW} // 성적을 상, 중, 하 세 단계로 분류

}

class Test {
    public static void main(String[] args) {

        Student3[] stuArr = {
                new Student3("나자바", true, 1, 1, 300),
                new Student3("김지미", false, 1, 1, 250),
                new Student3("김자바", true, 1, 1, 200),
                new Student3("이지미", false, 1, 2, 150),
                new Student3("남자바", true, 1, 2, 100),

                new Student3("안지미", false, 1, 2, 50),
                new Student3("황지미", false, 1, 3, 100),
                new Student3("강지미", false, 1, 3, 150),
                new Student3("이자바", true, 1, 3, 200),
                new Student3("나자바", true, 2, 1, 300),

                new Student3("김지미", false, 2, 1, 250),
                new Student3("김자바", true, 2, 1, 200),
                new Student3("이지미", false, 2, 2, 150),
                new Student3("남자바", true, 2, 2, 100),
                new Student3("안지미", false, 2, 2, 50),

                new Student3("황지미", false, 2, 3, 100),
                new Student3("강지미", false, 2, 3, 150),
                new Student3("이자바", true, 2, 3, 200)
        };

        System.out.printf("1. 단순그룹화(반별로 그룹화)%n");
        Map<Integer, List<Student3>> stubyBan = Stream.of(stuArr)
                .collect(groupingBy(Student3::getBan));

        for (List<Student3> ban : stubyBan.values()) {
            for (Student3 s : ban) {
                System.out.println(s);
            }
        }

        System.out.printf("%n2. 단순그룹화(성적별로 그룹화)%n");
        Map<Student3.Level, List<Student3>> stuByLevel = Stream.of(stuArr)
                    .collect(groupingBy(s -> {
                        if (s.getScore() >= 200) return Student3.Level.HIGH;
                    else if(s.getScore() >= 100) return Student3.Level.MID;
                    else                         return Student3.Level.LOW;
                    }));

        TreeSet<Student3.Level> keySet = new TreeSet<>(stuByLevel.keySet());

        for (Student3.Level key : keySet) {
            System.out.println("[" + key + "]");

            for (Student3 s : stuByLevel.get(key))
                System.out.println(s);
            System.out.println();
        }

        System.out.printf("%n3. 단순그룹화 + 통계(성적별 학생수)%n");
        Map<Student3.Level, Long> stuCntByLevel = Stream.of(stuArr)
                .collect(groupingBy(s -> {
                    if (s.getScore() >= 200) return Student3.Level.HIGH;
                else if (s.getScore() >= 100) return Student3.Level.MID;
                else                         return Student3.Level.LOW;
                }, counting()));
        for (Student3.Level key : stuCntByLevel.keySet())
            System.out.printf("[%s] - %d명, ", key, stuCntByLevel.get(key));
        System.out.println();

        System.out.printf("%n4. 다중그룹화(학년별, 반별)");
        Map<Integer, Map<Integer, List<Student3>>> stuByHakAndBan =
                Stream.of(stuArr)
                .collect(groupingBy(Student3::getHak,
                        groupingBy(Student3::getBan)
                ));

        for (Map<Integer, List<Student3>> hak : stuByHakAndBan.values()) {
            for (List<Student3> ban : hak.values()) {
                System.out.println();
                for (Student3 s : ban)
                    System.out.println(s);
            }
        }

        System.out.printf("%n5. 다중그룹화 + 통계(학년별, 반별1등)%n");
        Map<Integer, Map<Integer, Student3>> topStuByHakAndBan =
                Stream.of(stuArr)
                    .collect(groupingBy(Student3::getHak,
                            groupingBy(Student3::getBan,
                                    collectingAndThen(
                                            maxBy(comparingInt(Student3::getScore))
                                            , Optional::get
                                    )
                            )
                    ));

        for (Map<Integer, Student3> ban : topStuByHakAndBan.values())
            for (Student3 s : ban.values())
                System.out.println(s);

        System.out.printf("%n6. 다중그룹화 + 통계(학년별, 반별 성적그룹)%n");
        Map<String, Set<Student3.Level>> stuByScoreGroup = Stream.of(stuArr)
                .collect(groupingBy(s -> s.getHak() + "-" + s.getBan(),
                        mapping(s -> {
                            if (s.getScore() >= 200) return Student3.Level.HIGH;
                        else if (s.getScore() >= 100) return Student3.Level.MID;
                            else                       return Student3.Level.LOW;
                        }, toSet())
                ));

        Set<String> keySet2 = stuByScoreGroup.keySet();

        for (String key : keySet2) {
            System.out.println("[" + key + "]" + stuByScoreGroup.get(key));
        }
    }
}

```

- 결과

```
1. 단순그룹화(반별로 그룹화)
[나자바, 남, 1학년 1반, 300점]
[김지미, 여, 1학년 1반, 250점]
[김자바, 남, 1학년 1반, 200점]
[나자바, 남, 2학년 1반, 300점]
[김지미, 여, 2학년 1반, 250점]
[김자바, 남, 2학년 1반, 200점]
[이지미, 여, 1학년 2반, 150점]
[남자바, 남, 1학년 2반, 100점]
[안지미, 여, 1학년 2반,  50점]
[이지미, 여, 2학년 2반, 150점]
[남자바, 남, 2학년 2반, 100점]
[안지미, 여, 2학년 2반,  50점]
[황지미, 여, 1학년 3반, 100점]
[강지미, 여, 1학년 3반, 150점]
[이자바, 남, 1학년 3반, 200점]
[황지미, 여, 2학년 3반, 100점]
[강지미, 여, 2학년 3반, 150점]
[이자바, 남, 2학년 3반, 200점]

2. 단순그룹화(성적별로 그룹화)
[HIGH]
[나자바, 남, 1학년 1반, 300점]
[김지미, 여, 1학년 1반, 250점]
[김자바, 남, 1학년 1반, 200점]
[이자바, 남, 1학년 3반, 200점]
[나자바, 남, 2학년 1반, 300점]
[김지미, 여, 2학년 1반, 250점]
[김자바, 남, 2학년 1반, 200점]
[이자바, 남, 2학년 3반, 200점]

[MID]
[이지미, 여, 1학년 2반, 150점]
[남자바, 남, 1학년 2반, 100점]
[황지미, 여, 1학년 3반, 100점]
[강지미, 여, 1학년 3반, 150점]
[이지미, 여, 2학년 2반, 150점]
[남자바, 남, 2학년 2반, 100점]
[황지미, 여, 2학년 3반, 100점]
[강지미, 여, 2학년 3반, 150점]

[LOW]
[안지미, 여, 1학년 2반,  50점]
[안지미, 여, 2학년 2반,  50점]


3. 단순그룹화 + 통계(성적별 학생수)
[LOW] - 2명, [HIGH] - 8명, [MID] - 8명,

4. 다중그룹화(학년별, 반별)
[나자바, 남, 1학년 1반, 300점]
[김지미, 여, 1학년 1반, 250점]
[김자바, 남, 1학년 1반, 200점]

[이지미, 여, 1학년 2반, 150점]
[남자바, 남, 1학년 2반, 100점]
[안지미, 여, 1학년 2반,  50점]

[황지미, 여, 1학년 3반, 100점]
[강지미, 여, 1학년 3반, 150점]
[이자바, 남, 1학년 3반, 200점]

[나자바, 남, 2학년 1반, 300점]
[김지미, 여, 2학년 1반, 250점]
[김자바, 남, 2학년 1반, 200점]

[이지미, 여, 2학년 2반, 150점]
[남자바, 남, 2학년 2반, 100점]
[안지미, 여, 2학년 2반,  50점]

[황지미, 여, 2학년 3반, 100점]
[강지미, 여, 2학년 3반, 150점]
[이자바, 남, 2학년 3반, 200점]

5. 다중그룹화 + 통계(학년별, 반별1등)
[나자바, 남, 1학년 1반, 300점]
[이지미, 여, 1학년 2반, 150점]
[이자바, 남, 1학년 3반, 200점]
[나자바, 남, 2학년 1반, 300점]
[이지미, 여, 2학년 2반, 150점]
[이자바, 남, 2학년 3반, 200점]

6. 다중그룹화 + 통계(학년별, 반별 성적그룹)
[1-1][HIGH]
[2-1][HIGH]
[1-2][LOW, MID]
[2-2][LOW, MID]
[1-3][HIGH, MID]
[2-3][HIGH, MID]
```

### 스트림의 변환

- 정리

![image](https://user-images.githubusercontent.com/57219160/136909140-d014debc-baa8-496c-8e2b-2f5c88c67e9e.png)
![image](https://user-images.githubusercontent.com/57219160/136909172-c949e0c4-4936-4bdd-9b77-004b78092d0b.png)

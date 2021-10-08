---
title: 'Stream'
date: 2021-10-07 17:26:00
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

- RanDom() 함수의 범위

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

1. skip()

- 건너뛰기

2. limit()

- 잘라내기

3. distinct()

- 중복제거

4. filter()

- 걸러내기

5. sorted()

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

6. map()

- 변환

7. peek()

- forEach()와 비슷하지만 peek()는 중간연산, forEach()는 최종연산

8. flatmap()

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
  .distinct()                                   // Stream<String> -> Stream<String>, 대문자로 변경
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

- 작업 중간에 잘 진행되고 있는지 확인시 사용. 디버깅 용도

```
Stream<T> peek(Consumer<? super T> action) // 중간 연산(스트림의 요소를 소비x)
void      forEach(Consumer<? super T> action) // 최종 연산(스트림의 요소를 소비O)
```

- 아래처럼 여러작업을 수행할 떄 중간중간에 잘 진행되고 있는지 확인할 때 사용한다. 디버깅 용도

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

- [링크](https://highjune.dev/java/optional/)

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

## 조건 검사 - allMatch(), anyMatch(), noneMatch()

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

## 조건에 일치하는 요소 찾기 - findFirst(), findAny()

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

## 스트림의 요소를 하나씩 줄여가며 누적연산(accumulator) 수행 - reduce()

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

## 실습

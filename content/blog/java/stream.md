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

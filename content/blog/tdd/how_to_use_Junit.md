---
title: 'Junit5 사용하기'
date: 2023-04-13 21:54:00
category: 'TDD'
draft: false
---

> TDD를 사용하면서 여러 기본 활용법들을 기록한다.(너무 기본적인 것들은 제외)
- 나중에 깊게 공부하기 시작하면 각각의 게시물로 나누기
- [Introduction to AssertJ 문서](https://www.baeldung.com/introduction-to-assertj) 를 자주 찾아보자.
- [AssertJ Exception Assertions](https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#exception-assertion)

## BeforeEach

- 각각의 @Test, @RepeatedTest, @ParameterizedTest 등의 어노테이션이 붙은 테스트가 실행되기 전에 실행.
- [@BeforeEach](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/BeforeEach.html)
- 활용

```java
@BeforeEach
void setUp() {
	numbers = new HashSet<>();
	numbers.add(1);
	numbers.add(1);
	numbers.add(2);
	numbers.add(3);
}
```

## containsExactly()
- 해당 인자의 유무를 떠나서 순서까지 정확해야 함.

```java
@Test
@DisplayName("String을 split 했을시 배열 검증")
void split() {
	String[] result = "1,2".split(",");
	assertThat(result).containsExactly("1", "2");
}
```

## @ParameterizedTest
- 사용하기 위해서는 매개변수(실제값을 담은 argument라 칭함)를 넘겨주어야 하고, 공식 문서에서는 이를 ArgumentsProvider 라 부른다.
- 이 ArgumentProvider 로 사용될 수 있게 주석으로 만들어놓은 것이 argument source 들이고 ArgumentProvider로 사용할 수 있는 arguments source 는 @ArgumentsSoruce로 등록할 수 있다고 한다.
- argument source 들의 종류
	- ValueSource
	- MethodSource
	- CsvSource
	- 등등
- 활용1

```java
@ParameterizedTest
@DisplayName("Set 안에 포함된 요소들 확인하기 : @ViewSource ")
@ValueSource(ints = {1, 2, 3})
void contains(int input) {
	assertThat(numbers.contains(input)).isTrue();
}
```

- 활용2

```java
@ParameterizedTest
@DisplayName("Set 안에 포함된 요소들 확인하기 : @CsvSource ")
@CsvSource(value = {"1:true", "2:true", "3:true"}, delimiter = ':')
void contains(int input, boolean expected) {
	assertThat(numbers.contains(input)).isEqualTo(expected);
}
```

- 활용3

```java
@ParameterizedTest
@DisplayName("Set 안에 포함된 요소들 확인하기(예제) : @CsvSource(true, false 둘 다 사용해보기)")
@CsvSource(value = {"test:test", "tEst:test", "Java:java"}, delimiter = ':')
void contains(String input, String expected) {
	String actual = input.toLowerCase();
	assertThat(actual).isEqualTo(expected);
}
```

- [ParameterizedTest 관련 badeldung문서](https://www.baeldung.com/parameterized-tests-junit-5)



## 예외확인

```java
@Test
@DisplayName("유효하지 않은 위치의 스트링 값을 가져올 때 에러가 발생할 것이다.")
void stringCharAt() {
	String problem = "abc";

	// #1
	assertThatThrownBy(() -> problem.charAt(problem.length()))
			.isInstanceOf(StringIndexOutOfBoundsException.class)
			.hasMessageContaining("String index out of range: 3");

	// #2
	assertThatExceptionOfType(StringIndexOutOfBoundsException.class)
			.isThrownBy(() -> problem.charAt(problem.length()))
			.withMessageMatching("String index out of range: \\d");
}
```

- [Exception Assertions guide](https://joel-costigliola.github.io/assertj/assertj-core-features-highlight.html#exception-assertion)

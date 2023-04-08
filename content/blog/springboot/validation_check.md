---
title: '컨트롤러 이외에서 파라미터 유효성 검증(Validation)'
date: 2023-02-03 23:47:13
category: 'springboot'
draft: false
---

- 우리는 컨트롤러단에서 @Valid 로써 필드에 @NotNull, @Size, @Pattern 등을 통해 필드의 유효성을 검증한다.
- 하지만 경우에 따라 컨트롤러에서 바로 해당 객체를 검증하지 않고 별도로 검증해야 하는 순간들이 있다.
    - 하나의 컨트롤러로 여러 객체들을 받은 후에 검증을 해야하는 경우 등.

- 그럴 때는 `javax.validation.*`의 객체를 사용한다.


- validationCheckForRequestBodyDto 함수
```java
	public class ValidationCheckTest {

		//	(..중략..)

		private void validationCheckForRequestBodyDto(Object object) {
			ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
			Validator validator = factory.getValidator();
			Set<ConstraintViolation<Object>> violations = validator.validate(object);
			StringBuilder stringBuilder = new StringBuilder();
			String field = "";
			String defaultMessage = "";
			if (violations.size() > 0) {
				for (ConstraintViolation<Object> violation : violations) {
					stringBuilder.append(String.format("%s : %s \r\n", violation.getInvalidValue(), violation.getMessage()));
					field = violation.getPropertyPath().toString();
					defaultMessage = violation.getMessage();
				}
				log.warn(stringBuilder.toString());
				FieldError fieldError = new FieldError(SingleDto.CreateRequest.class.getName(), field, defaultMessage);
				throw new BindingException(fieldError);
			}
		}

		// 밑에서 동작할 테스트 함수
		@Test
		public void validationCheckTest() {
			// 생략

		}

	}
```


- 테스트할 Dto 객체들

```java
@Data
	static class MemberDto {
		@NotNull(message = "MemberDto.name 값은 반드시 필요합니다.")
		private String name;
		@NotNull(message = "MemberDto.home 값은 반드시 필요합니다.")
		private String home;
	}

	@Data
	static class ItemDto {
		@NotNull(message = "ItemDto.company 값은 반드시 필요합니다.")
		private String company;
		@NotNull(message = "ItemDto.price 값은 반드시 필요합니다.")
		private Integer price;
	}
```

- 테스트 코드

```java
public class ValidationCheckTest {
	// (..중략..)

		private void validationCheckForRequestBodyDto(Object object) {
			// (..생략..)
		}

	
		@Test
		public void validationCheckTest() {
			MemberDto memberDto = new MemberDto();
			memberDto.setHome("seoul");    // name 값도 필수값인데 넣지 않았음.
			Assertions.assertThatThrownBy(() -> this.validationCheckForRequestBodyDto(memberDto))
					.isInstanceOf(BindingException.class);
	
			ItemDto itemDto = new ItemDto();
			itemDto.setCompany("google");    // price 값도 필수값인데 넣지 않았음.
			Assertions.assertThatThrownBy(() -> this.validationCheckForRequestBodyDto(itemDto))
					.isInstanceOf(BindingException.class);
		}

}
```

- validationCheckTest() 함수 실행 후 로그

```markdown
2023-04-07 14:42:21 WARN  EtcTest.validationCheckForRequestBodyDto - null : MemberDto.name 값은 반드시 필요합니다. 

2023-04-07 14:42:21 WARN  EtcTest.validationCheckForRequestBodyDto - null : ItemDto.price 값은 반드시 필요합니다.
```

- Validator 함수
    
    ```markdown
    Set<ConstraintViolation<Object>> violations = validator.validate(검증할 객체);
    ```
    
    - `<ConstraintViolation<Object>>`  violation 의 함수
        - getInvalidValue()
            - 유효하지 않은 값. 필수값이지만 안 넣었다면 null과 같은 값
        - getMessage()
            - 유효성 체크에 넣은 message 또는 default 메시지

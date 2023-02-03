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

- 예를 들어 아래와 같은 Dto 객체가 있다고 하자.
```java
public class UserDto {

    public static class UserRequest {
        
        @NotEmpty(message = "이름 정보는 필수입니다.")
        private String name;

        @NotEmpty(message = "이메일 정보는 필수입니다.")
        private String email;

        ...(중략)...
    }

}
```

- 위의 객체를 아래처럼 별도로 유효성 검증을 할 수 있다.
```java

public void checkValidation () {
    
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        Set<ConstraintViolation<RequestDto.Member>> violations = validator.validate(memberDto);
        StringBuilder stringBuilder = new StringBuilder();

        for (ConstraintViolation<RequestDto.Member> violation : violations) {
            stringBuilder.append(String.format("%s : %s \r\n", violation.getInvalidValue(), violation.getMessage()));
        }
        
        if (violations.size() > 0) {
            log.error(stringBuilder.toString());
            throw new CommonException(Errors.GENERAL_WRONG_PARAM);
        }
}
    
```

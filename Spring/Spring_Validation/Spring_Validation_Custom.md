# Validation

# Validation

## 문제

- 데이터 유효성 검사 로직의 문제점

  ![Untitled](Spring_Validation_Custom/Untitled.png)

  1. 애플리케이션 전체에 분산되어 있다.
  2. 코드 중복이 심하다.
  3. 비즈니스 로직에 섞여있어, 검사 로직 추적이 어렵고 애플리케이션이 복잡해진다.

## 해결 방법

- Java에서 Bean Validation이라는 데이터 유효성 검사 프레임워크를 제공한다.
- 위 문제들을 해결하기 위해 다양한 제약(Contraint)을 도메인 모델(Domain Model)에 어노테이션(Annotaion)으로 정의할 수 있게 한다.
- 유효성 검사가 필요한 객체에 직접 정의하는 방법으로 기존 유효성 검사 로직의 문제점을 해결한다.

  ![Untitled](Spring_Validation_Custom/Untitled%201.png)

## 제약 검사 설정과 기능

- Validation Starter를 추가한다.
- Service나 Bean에서 사용하기 위해서는 @Validate와 @Valid를 추가해야 한다.
  - @Valid가 설정된 메소드가 호출될 때 유효성 검사를 진행한다.
  - Controller에서는 @Validated가 필요 없이 검사가 필요한 곳에 @Valid를 추가하면 된다.
- Bean Validation 에서는 @Length, @NotBlank, @NotNull 등을 이용하여 설정할 수 있다.

```java
public class CreateContact{
	@Length(max=64)
	@NotBlank
	private String uid;

	@NotNull
	private ContactType contactType;

	@Length(max=1_600)
	private String contact;

}
```

- 유효성 검사 진행 시 같은 데이터를 여러 번 실행하게 될 경우 애플리케이션 성능에 영향을 미칠 수 있기 때문에 주의해야 한다.
- Bean Validation에서 제공하는 제약이 아닌, 필요한 제약을 직접 정의해서 사용할 수 있다.

# Custom Annotation을 이용하여 유효성 체크 정리

1. 커스텀 어노테이션 정의
2. ConstraintValidator를 구현한 클래스 선언
   1. ConstraintValidator<커스텀 어노테이션, 검증할 객체>
3. 해당 객체에 @Valid 붙여주기 & 해당 객체 안에서 검증할 필드에 커스텀 어노테이션 작성하기

## 1. Custom Annotation 정의

```java
@Documented
@Constraint(validatedBy = PasswordValidation.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Password {

		// 유효성 체크 시 잘못된 경우 보여줄 메시지
    String message() default "암호 오류";

		// 유효성 검사기 어떠한 상황에서 이루어져야 하는지 정의
    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    int min() default 8;

    int max() default 12;

}
```

- @Documented

  - javadoc으로 api 문서 생성 시 설명도 포함하도록 지정하기 위해 선언

- @Constraint

  - 인터페이스 ConstraintValidator를 구현한 객체를 지정한다.
  - 해당 객체를 검증한다는 것을 표기한다.

- @Target

  - 어노테이션을 어느 곳에 적용할 것인지 결정
  - default는 모든 곳이다.
  - 인스턴스 변수에만 적용 → ElementType.FIELD
  - 클래스, 인터페이스, enum에만 적용 → ElementType.TYPE
  - 메소드에만 적용 → ElementType.METHOD
  - 등이 존재한다.

- @Retention
  - 어노테이션이 언제까지 유지될 것인지 결정.
  - RetentionPolicy.SOURCE
    - 컴파일 후 정보들이 사라진다. @Override, @SuppressWarnings 등
  - RetentionPolicy.CLASS
    - default 값.
    - 컴파일 타임 까지만 존재하기 때문에 .class 파일에는 존재하지만, 런타임시 없어진다
    - Reflection 사용이 불가능하다.
  - RetentionPolicy.RUNTIME
    - 런타임시 까지 존재한다.
    - 커스텀 어노테이션 만들 때 주로 사용한다.
    - Reflection 사용이 가능하다.

## 2. ConstraintValidator 구현 클래스

```java
public class PasswordValidation implements ConstraintValidator<Password,String> {

    private int min;
    private int max;

    @Override
    public void initialize(Password constraintAnnotation) {
        min=constraintAnnotation.min();
        max=constraintAnnotation.max();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        int passwordLength = value.length();
        return (!value.isEmpty() && min <= passwordLength && passwordLength <= max);
    }

}
```

- ConstraintValidator<어노테이션 이름, 검증할 객체>

- initialize 메소드를 오버라이딩 하여 초기화를 진행한다.

- isValid 메소드를 오버라이딩 하여 실제 객체를 검증한다.
  - true인 경우 정상 객체
  - false인 경우 비정상 객체

## 3. 검증

```java
@RestController
@RequestMapping("/api/user")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping
    public ResponseEntity<ResponseDto> createUser(@RequestBody @Valid UserRequestDto requestDto) throws UserPresentException {
        return userService.createUser(requestDto);
    }
}
```

- @Valid가 붙은 객체에 있는 값들의 유효성을 체크한다.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserRequestDto {

    @NotBlank
    private String id;

    @NotBlank
    private String name;

    @Password
    private String password;

    @NotBlank
    private String phoneNumber;

}
```

- @Valid 객체 안 값들 중 유효성 체크가 필요한 부분에 커스텀 어노테이션을 붙인다.

## 4. 예외 처리

```java
@ControllerAdvice
public class ExceptionController {

    @ExceptionHandler({MethodArgumentNotValidException.class})
    public ResponseEntity<ResponseDto> notValidException(MethodArgumentNotValidException ex){
        ResponseDto responseDto = ResponseDto.builder()
                .message(ex.getBindingResult().getAllErrors().get(0).getDefaultMessage())
                .build();
        return new ResponseEntity<>(responseDto, HttpStatus.BAD_REQUEST);
    }
}
```

- @Valid가 붙은 객체에 있는 값들의 유효성에서 문제가 있는 경우 MethodArgumentNotValidException 예외가 발생한다.
- getBindingResult().getAllErrors().get(0).getDefaultMessage()를 통해 Custom Annotation의 message를 가져올 수 있다.

출처

[https://meetup.toast.com/posts/223](https://meetup.toast.com/posts/223)

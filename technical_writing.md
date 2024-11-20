# API 에러 응답 problem detail: 스프링에서 사용하기
이 문서는 spring 6, java 17 이상의 버전을 기준으로 합니다.  

## 기존의 예외 처리 방식의 문제점
스프링을 사용해서 서버의 예외를 처리하기 위해선, 전역적으로 예외를 처리해주는 ExceptionHandler를 주로 구현하여 사용합니다. 이는 아래와 같습니다.
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler
    public ResponseEntity<ResponseForError> handleException(Exception e) {
        ResponseForError responseForError = new ResponseForError(e.getMessage(), HttpStatus.BAD_REQUEST.value());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(responseForError);
    }
}

public record ResponseForError(
        String message,
        String status
) {
}
```
위와 같은 방식은 ResponseForError 클래스 같은 오류 응답 형식을 정해야 합니다. 이에 대한 고민을 줄이기 위해 RFC에서 정의한 오류 응답 표준이 존재합니다.

## API 예외 표준: RFC 9457
RFC 9457에선 HTTP API에 대한 새로운 오류 응답 형식을 정의할 필요가 없도록 HTTP 응답에 오류 세부 정보를 전달하는 방법으로 'problem detail'을 정의합니다.

problem detail은 아래와 같은 구성 요소를 가지고 있습니다.

- type: 문제 유형을 식별하는 URI입니다. 설정하지 않은 경우 기본값은 "about:blank"입니다.
- status: 응답 상태 코드의 숫자입니다.
- title: 문제 유형에 대한 간단한 요약입니다.
- detail: 문제 원인에 대한 자세한 설명입니다.
- instance: 문제의 원인을 식별하는 URI입니다.
- properties: 문제에 대한 추가적인 정보를 제공하는 필드입니다.

예시 <br/>
```
HTTP/1.1 404 Not Found
Content-Type: application/json
{
    "type": "about:blank",
    "title": "Not Found",
    "status": 404,
    "detail": "존재하지 않는 대시보드입니다.",
    "instance": "/v1/processes"
}
```

## ProblemDetail
스프링은 RFC 9457에서 정의된 'problem detail'을 구현한 ProblemDetail 클래스가 존재합니다.

ProblemDetail 클래스은 아래와 같은 필드로 구성되어 있습니다.
```java
public class ProblemDetail {

	private static final URI BLANK_TYPE = URI.create("about:blank");


	private URI type = BLANK_TYPE;

	@Nullable
	private String title;

	private int status;

	@Nullable
	private String detail;

	@Nullable
	private URI instance;

	@Nullable
	private Map<String, Object> properties;
}
```
각 필드에는 getter, setter가 존재합니다. 단일 프로퍼티를 추가할 수 있는 setProperty 메서드도 존재합니다.

### 사용법
ProblemDetail 클래스는 정적 팩토리 메서드로 인스턴스를 생성할 수 있습니다.
```java
public static ProblemDetail forStatus(HttpStatusCode status) {
	Assert.notNull(status, "HttpStatusCode is required");
	return forStatus(status.value());
}

public static ProblemDetail forStatus(int status) {
	return new ProblemDetail(status);
}

public static ProblemDetail forStatusAndDetail(HttpStatusCode status, String detail) {
	Assert.notNull(status, "HttpStatusCode is required");
	ProblemDetail problemDetail = forStatus(status.value());
	problemDetail.setDetail(detail);
	return problemDetail;
}
```
이를 ControllerAdvice에 적용하면 아래와 같은 형식으로 사용할 수 있습니다.
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler
    public ResponseEntity<ProblemDetail> handleException(Exception e) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, e.getMessage());
        return ResponseEntity.of(problemDetail).build();
    }
}
```
instance와 type 필드는 설정하지 않은 경우 각각 요청의 URI, 상태 코드의 메시지가 값이 됩니다.

## ErrorResponse
ErrorResponse는 RFC 9457에서 정의한 오류 응답 전체를 나타내는 인터페이스입니다. 이 인터페이스는 세 가지 주요 메서드가 존재합니다.
```java
	HttpStatusCode getStatusCode();

	default HttpHeaders getHeaders() {
		return HttpHeaders.EMPTY;
	}

	ProblemDetail getBody();
```
스프링 MVC에서 정의된 예외는 이 인터페이스를 구현하고 있습니다.

### 사용자 커스텀 예외를 ErrorResponse를 구현하여 생성하는 방법
이 인터페이스를 사용하여 사용자 커스텀 예외를 생성할 수 있습니다.
```java
public class CustomException extends RuntimeException implements ErrorResponse {
    
    private final HttpStatus httpStatus;
    private final ProblemDetail problemDetail;
    
    CustomException(String message, HttpStatus status) {
        super(message);
        this.httpStatus = status;
        this.problemDetail = ProblemDetail.forStatusAndDetail(status, message);
    }
    
    @Override
    public HttpStatusCode getStatusCode() {
        return httpStatus;
    }

    @Override
    public ProblemDetail getBody() {
        return problemDetail;
    }
}

@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler
    public ResponseEntity<ProblemDetail> handleCustomException(CustomException e) {
        return ResponseEntity.of(e.getBody()).build();
    }
}
```

### ErrorResponseException
스프링에서는 ErrorResponse의 기본 구현체인 ErrorResponseException을 제공하고 있습니다. 이를 사용하여 더 간단하게 ProblemDetail 클래스를 필드로 가지는 커스텀 예외를 생성할 수 있습니다.
```java
public class ErrorResponseException extends NestedRuntimeException implements ErrorResponse {

	private final HttpStatusCode status;

	private final HttpHeaders headers = new HttpHeaders();

	private final ProblemDetail body;

	private final String messageDetailCode;

	@Nullable
	private final Object[] messageDetailArguments;

	public ErrorResponseException(
			HttpStatusCode status, ProblemDetail body, @Nullable Throwable cause,
			@Nullable String messageDetailCode, @Nullable Object[] messageDetailArguments) {
		super(null, cause);
		this.status = status;
		this.body = body;
		this.messageDetailCode = initMessageDetailCode(messageDetailCode);
		this.messageDetailArguments = messageDetailArguments;
	}
}
```
이 클래스는 getHeaders(), getStatus(), getBody() 메서드를 오버라이딩 하고 있습니다.

이 클래스를 상속하여 예외를 구현하면 위 메서드들을 재정의할 필요가 없다는 장점이 있으며, set 메서드를 활용하여 ProblemDetail을 설정할 수도 있습니다.
```java
public class CustomException extends ErrorResponseException {
    
    CustomException(String message, HttpStatus status) {
        super(status);
        setDetail(message);
    }
}
```
위처럼 더 간단하게 커스텀 예외를 생성할 수 있습니다.

## ResponseEntityExceptionHandler: 스프링에서 기본적으로 제공해주는 예외 처리 클래스
스프링 MVC는 @ControllerAdvice의 기본 클래스로 간편하게 사용할 수 있는 ResponseEntityExceptionHandler 클래스를 제공합니다.

이 클래스는 모든 스프링 MVC 예외와 특정 자바 예외를 처리하는 @ExceptionHandler 메서드를 구현하고 있습니다. 이 메서드는 ResponseEntity<ProblemDetail> 타입을 반환합니다.
```java
@ExceptionHandler({
			HttpRequestMethodNotSupportedException.class,
			HttpMediaTypeNotSupportedException.class,
			HttpMediaTypeNotAcceptableException.class,
			MissingPathVariableException.class,
			MissingServletRequestParameterException.class,
			MissingServletRequestPartException.class,
			ServletRequestBindingException.class,
			MethodArgumentNotValidException.class,
			HandlerMethodValidationException.class,
			NoHandlerFoundException.class,
			NoResourceFoundException.class,
			AsyncRequestTimeoutException.class,
			ErrorResponseException.class,
			MaxUploadSizeExceededException.class,
			ConversionNotSupportedException.class,
			TypeMismatchException.class,
			HttpMessageNotReadableException.class,
			HttpMessageNotWritableException.class,
			MethodValidationException.class,
			BindException.class
		})
@Nullable
public final ResponseEntity<Object> handleException(Exception ex, WebRequest request) throws Exception {
    ...
}
```
이를 상속하여 간편하게 예외 처리를 할 수 있으며, 별개의 예외에 대하여 메서드를 재정의해 사용할 수 있습니다.
```java
@ControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException e,
            HttpHeaders headers,
            HttpStatusCode status,
            WebRequest request
    ) {
        ...
    }
}
```

### 예외 로그 추가
예외 처리가 중요한 만큼, 예외 발생 시 확인할 수 있어야 합니다. 예외를 확인하기 위한 방식으로 로그를 사용합니다.

예외 로그를 추가하기 위해선 handleException() 메서드에 로그 처리 메서드를 사용하는 것을 생각해 볼 수 있습니다.

하지만 handleException()은 final 메서드이기 때문에 재정의하여 사용할 수 없습니다. 이때 상속과 조합을 활용할 수 있습니다.

```java
@Component
public class UnexpectedExceptionHandler extends ResponseEntityExceptionHandler {

}

@Slf4j
@ControllerAdvice
@RequiredArgsConstructor
public class GlobalExceptionHandler {

    private final UnexpectedExceptionHandler handler;

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleUnexpectedException(Exception e, WebRequest request) {
        ProblemDetail problemDetail = handleException(e, request);
        log.error(problemDetail);
        return ResponseEntity.of(problemDetail).build();
    }

    private ProblemDetail handleException(Exception e, WebRequest request) {
        try {
            return (ProblemDetail) handler.handleException(e, request).getBody();
        } catch (Exception ex) {
            return ProblemDetail.forStatusAndDetail(
                    HttpStatus.INTERNAL_SERVER_ERROR,
                    "예기치 못한 오류가 발생하였습니다."
            );
        }
    }
```

## 참고
- RFC 9457: https://www.rfc-editor.org/rfc/rfc9457 <br/>
- Spring Rest Exception 설명: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html

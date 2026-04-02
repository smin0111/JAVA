# Java Study Notes (자바의 정석 4판 8장 정리)

## 개요

자바의 정석 8장 예외처리와 실무 확장 개념을 정리한 스터디 노트다.
기본 예외처리 문법뿐만 아니라, Blue 백엔드 프로젝트에서 실제로 적용했던 커스텀 예외와 GlobalExceptionHandler 구조를 매칭하여 실무 적용 방식까지 전부 포함했다.

---

# 1. 프로그램 오류 종류

- 컴파일 에러: 문법 오류, 실행 불가
- 런타임 에러: 실행 중 오류, 예외처리 가능
- 논리 에러: 결과 이상, 사람이 직접 디버깅해서 수정

우리가 자바에서 코드로 다루는 핵심은 '런타임 에러'를 안전하게 처리하는 것이다.

---

# 2. 런타임 에러 구분 (Error vs Exception)

## Error
메모리 부족(OutOfMemoryError), 스택오버플로우 등 JVM 자체의 심각한 문제다.
개발자가 코드로 수습할 수 없으므로 신경 쓰지 않아도 무방하다.

## Exception (중요)
개발자가 코드로 수습하고 처리할 수 있는 오류다.
크게 RuntimeException과 Checked Exception으로 나뉜다.

---

# 3. Runtime vs Checked

## RuntimeException (언체크 예외)
컴파일러가 예외 처리를 강제하지 않는다(체크 X). 주로 프로그래머의 실수로 발생한다.
예: NullPointerException, IndexOutOfBoundsException

## Checked Exception (체크 예외)
컴파일러가 "너 이거 예외처리 안 하면 컴파일 안 해줌"이라고 강제한다(체크 O). 주로 외부 환경 문제로 발생한다.
예: IOException, SQLException

---

# 4. try-catch 기본과 finally

```java
try {
    // 에러가 날 수도 있는 위험한 코드
} catch(Exception e) {
    // 예외가 터지면 여기로 넘어와서 수습하는 흐름
} finally {
    // 에러가 나든 안 나든, return이 있든 무조건 마지막에 실행됨 (주로 자원 반납용)
}
```

---

# 5. throw vs throws

- throw: 내가 내 손으로 직접 예외를 일부러 '발생'시킬 때 쓴다.
- throws: 메서드 선언부에 적어서, 이 메서드를 호출한 쪽으로 예외 처리를 '떠넘김(전파)'할 때 쓴다.

---

# 6. 예외 전파와 위치 (실무 핵심)

메서드 안에서 예외를 삼켜서 처리 안 하면, 나를 부른 상위 계층으로 계속 올라간다.
(Repository -> Service -> Controller -> 메인)

나쁜 구조: 모든 계층에서 매번 try-catch로 단편적으로 예외를 빨아들이는 것.
좋은 구조: 예외는 한 곳(글로벌)에서만 모아서 일괄 처리한다.

- Repository, Service, Controller: 예외 발생 시 상위로 전파(throws)
- GlobalHandler: 전파된 모든 예외를 최종적으로 받아서 일괄 처리

---

# 7. 사용자 정의 예외 (실무 핵심)

자바가 제공하는 기본 Exception 만으로는 이게 도대체 무슨 의미의 비즈니스 에러인지 프론트가 알 길이 없다.
그래서 RuntimeException을 상속받아 우리 프로젝트만의 예외 클래스를 직접 만들어서 쓴다. 

(Blue 백엔드 작성 예시: 부모/자식 커스텀 예외 구조 만들기)
```java
// 우리 프로젝트 공통 비즈니스 부모 예외
class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}

// 회원 조회 관련 상세 자식 예외
class UserNotFoundException extends BusinessException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```
실무에서는 이처럼 의미 불명인 RuntimeException("오류") 대신 무조건 UserNotFoundException과 같은 커스텀 예외를 만들어서 쓴다.

---

# 8. 실무 예외 응답 JSON 설계 및 글로벌 핸들러

프론트엔드와 통신 시 약속된 규격으로 에러 응답(JSON)을 내려주는 것이 백엔드 API 실무의 정석이다.

목표 응답 JSON 구조:
```json
{
  "timestamp": "2026-04-02T12:00:00",
  "status": 404,
  "error": "USER_NOT_FOUND",
  "message": "회원이 존재하지 않습니다",
  "path": "/users/1"
}
```

(Blue 백엔드 작성 예시: Spring 글로벌 예외 처리기)
```java
// 스프링에서 모든 컨트롤러의 예외를 중앙 제어하는 핸들러
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 내가 만든 BusinessException이 전파되어 올라오면 여기가 최종적으로 낚아챔
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        ErrorResponse response = new ErrorResponse(404, "USER_NOT_FOUND", e.getMessage());
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
    }
}
```
결론적으로 Service 계층에서는 `throw new UserNotFoundException("회원이 존재하지 않습니다");` 한 줄만 툭 던지면, 코드 제어권이 GlobalExceptionHandler로 넘어가서 알아서 JSON 규격으로 포장해 프론트로 쏴주는 구조다.

---

# 9. try-with-resources 와 예외 체이닝

- try-with-resources: 실무에서는 finally에서 일일이 close()를 부르지 않고, try 괄호 `()` 안에 파일이나 자원을 선언하면 알아서 깔끔하게 닫히게 만든다.
- 예외 체이닝: DB 에러(SQLException)가 터졌을 때, 이 근본 원인(cause) 예외를 품은 채로 감싸서 내 커스텀 예외로 던지는 필수 패턴이다.

(Blue 백엔드 작성 예시: 제미나이 등 외부 API 통신 예외 체이닝)
```java
try {
    // 외부 AI 서버 연동 통신 로직 (네트워크 에러, 타임아웃 발생 가능성)
} catch(IOException e) {
    // 실제 발생한 외부망 에러(e)를 원인으로 보존한 채, 우리 백엔드 전용 예외로 포장해서 던짐
    throw new ExternalApiException("AI 서버 통신 실패", e);
}
```

---

# 10. if vs Exception 분기 기준 (면접 단골)

if문 사용 (값의 유효성 검사, 정상적인 비즈니스 방어 로직):
- 비밀번호 길이 8자리 미만 입력
- 통장 잔액 부족 결제 시도
- 이메일 형식 정규식 위반

Exception 사용 (심각한 시스템 이슈, 외부망 오류 차단 로직):
- DB에서 회원 조회 실패 시 (데이터 정합성 문제)
- DB 커넥션 다운
- 외부 AI API 연동 서버 다운, 타임아웃

한 줄 정리: if는 값 유효성 검사용, Exception은 심각한 프로그램 보호 및 오류 전파용.

---

# 최종 요약 (실무 핵심 7 계명)

1. RuntimeException 기반 무조건 내 커스텀 예외로 감싸서 사용한다.
2. 예외는 중간에 잡고 뭉개지 말고 위로 끝까지 전파 후 한 곳에서 처리한다.
3. 그 집중 처리소가 바로 Spring의 GlobalExceptionHandler (@RestControllerAdvice) 다.
4. 자원 반납은 무조건 try-with-resources 를 돌린다.
5. 에러를 덮어씌울 땐 원본 에러(cause)를 꼭 품는 예외 체이닝을 쓴다.
6. 단순 값 검증은 if, 시스템 에러 상황은 Exception 으로 나눈다.
7. 프론트엔드가 파싱하기 편하게 에러 응답은 무조건 표준화된 ErrorCode Enum과 JSON 규격으로 내려준다.

# Java Study Notes (자바의 정석 4판 9장 정리)

## 개요

자바의 정석 9장은 java.lang 패키지 파트다.
우리가 자바 공부하면서 계속 써왔던 String, System.out.println(), Math.random(), Integer.parseInt() 같은 것들이 전부 여기서 다루는 클래스들이다.

---

# 1. Object 클래스

자바에서 만든 모든 클래스는 자동으로 Object라는 최상위 부모를 가진다.
class Dog {} 라고 선언해도 내부적으로는 class Dog extends Object {} 로 처리된다.

핵심 메서드:

| 메서드 | 의미 |
|---|---|
| toString() | 객체를 텍스트로 설명 |
| equals() | 두 객체 내용 비교 |
| hashCode() | 객체의 해시값 반환 |

## toString()

객체를 그냥 출력하면 "클래스명@메모리주소" 같은 이상한 값이 나온다.
사람이 읽기 좋게 하려면 직접 오버라이딩해야 한다.

```java
class Person {
    String name = "철수";

    @Override
    public String toString() {
        return name;
    }
}
Person p = new Person();
System.out.println(p); // "철수" 출력
```

(Blue 백엔드 작성 예시: DTO 로그 확인용)
```java
// MemberDTO에 toString 재정의해두면 로그 찍을 때 주소값 대신 실제 회원 데이터가 보임
@Override
public String toString() {
    return "MemberDTO{memberId='" + memberId + "', name='" + name + "'}";
}
```

## equals()

== 는 주소(참조)를 비교하고, equals() 는 내용(값)을 비교한다.
문자열 비교는 무조건 equals() 를 써야 한다.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);      // false (주소 다름)
System.out.println(a.equals(b)); // true  (내용 같음)
```

(Blue 백엔드 작성 예시: JWT role 체크, 비밀번호 검증)
```java
if("ADMIN_USER".equals(memberRole)) {
    // 관리자 권한 허용
}

if(inputPassword.equals(savedPassword)) {
    // 로그인 성공 처리
}
```

---

# 2. String 클래스

## 불변(immutable) 특성

String은 한 번 만들어지면 내부 값이 절대 변하지 않는다.
s += " World" 처럼 이어 붙이면 기존 객체가 수정되는 게 아니라 새 객체가 생성되고 s가 새 주소를 가리키게 된다.

## 자주 쓰는 메서드

```java
String s = "Hello World";

s.length();            // 길이 (11)
s.substring(0, 5);     // 일부 추출 ("Hello")
s.contains("World");   // 포함 여부 (true)
s.replace("Hello", "Hi"); // 치환
s.toUpperCase();       // 대문자 변환
s.trim();              // 앞뒤 공백 제거
s.split(" ");          // 분리해서 배열 반환
s.startsWith("Hello"); // 특정 문자로 시작하는지 (true)
```

(Blue 백엔드 작성 예시: Authorization 헤더에서 JWT 토큰 추출)
```java
String header = request.getHeader("Authorization");
if (header != null && header.startsWith("Bearer ")) {
    String token = header.substring(7); // "Bearer " 7글자 제거 후 토큰만 추출
    jwtUtil.validateToken(token);
}
```

---

# 3. StringBuilder

String은 불변이라 반복 문자열 조합 시 성능이 나빠진다.
반복문 안에서 문자열을 조합할 때는 StringBuilder를 쓴다.

```java
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(", ");
sb.append("World");
String result = sb.toString(); // "Hello, World"
```

규칙: 문자열 조금만 붙일 때는 String, 반복문 안에서 많이 붙일 때는 StringBuilder.

(Blue 백엔드 작성 예시: AI 응답 메시지 조합)
```java
StringBuilder response = new StringBuilder();
for (String intentResult : intentResults) {
    response.append(intentResult).append("\n");
}
return response.toString();
```

---

# 4. Math 클래스

static 메서드만 있어서 new 없이 Math.메서드명() 으로 바로 호출한다.

```java
Math.random();       // 0.0 이상 1.0 미만 랜덤 실수
Math.abs(-5);        // 절대값 (5)
Math.round(3.6);     // 반올림 (4)
Math.ceil(3.1);      // 올림 (4.0)
Math.floor(3.9);     // 내림 (3.0)
Math.max(10, 20);    // 더 큰 값 (20)
Math.pow(2, 3);      // 거듭제곱 (8.0)

// 0~9 사이 랜덤 정수
int r = (int)(Math.random() * 10);
```

---

# 5. Wrapper 클래스

## 왜 필요한가?

List, Map 같은 컬렉션에는 기본형(int, double 등)을 직접 넣을 수 없고 객체만 들어간다.
그래서 기본형을 객체로 포장한 Wrapper 클래스가 존재한다.

```java
List<int> list = new ArrayList<>();     // 컴파일 에러
List<Integer> list = new ArrayList<>();  // 가능
```

| 기본형 | Wrapper |
|---|---|
| int | Integer |
| double | Double |
| boolean | Boolean |
| char | Character |
| long | Long |

## 오토박싱 / 언박싱

자바 5부터 기본형과 Wrapper 클래스 사이를 자동으로 변환해준다.

```java
Integer a = 10; // 오토박싱 (int -> Integer)
int b = a;      // 언박싱 (Integer -> int)
```

## 가장 많이 쓰는 기능: 문자열 -> 숫자 변환

API나 사용자 입력에서 받은 데이터는 항상 문자열이라 숫자로 변환이 필요하다.

```java
String ageStr = "25";
int age = Integer.parseInt(ageStr);

String priceStr = "3.14";
double price = Double.parseDouble(priceStr);
```

(Blue 백엔드 작성 예시: 토스 결제 금액 위변조 검증)
```java
// 토스에서 넘어온 결제 금액은 문자열이라 Long으로 변환 후 비교
String amountStr = tossResponse.get("amount");
long amount = Long.parseLong(amountStr);
if (amount != expectedAmount) {
    throw new BusinessException("결제 금액 위변조 감지");
}
```

---

# 최종 정리

- Object: 모든 클래스의 최상위 부모. toString()은 로그용으로 오버라이딩, equals()는 내용 비교 시 필수.
- String: 불변 객체. 문자 비교는 equals(), 반복 조합은 StringBuilder.
- Math: static 메서드 도구 모음. new 없이 바로 호출.
- Wrapper: 기본형을 객체로 포장. 컬렉션 사용 시, parseInt() 등 문자->숫자 변환 시 필수.

다음 학습 노트에서는 11장 컬렉션 프레임워크(List, Map, Set)로 이어진다.

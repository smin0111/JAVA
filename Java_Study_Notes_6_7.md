# Java Study Notes (자바의 정석 4판 6~7장 정리)

## 개요

자바의 정석 4판 기준 Chapter 06, 07 학습 기록.
이 구간은 단순 자바 문법을 넘어 객체지향 프로그래밍(OOP)의 실질적인 뼈대를 잡는 핵심 파트다.
이 두 장을 이해하는 것은 자바 문법 사용자에서 객체지향 개발자로 넘어가는 중요한 관문이다.

---

# Chapter 06. 객체지향 프로그래밍 I

4판 기준 6장에서는 객체와 클래스의 기본 개념, 변수와 메서드의 생성 원리, 그리고 생성자와 초기화 방식 등을 다룬다.

## 1. 객체지향이 필요한 이유

기존 절차지향 프로그래밍 방식은 코드가 커질수록 꼬이기 쉽고, 재사용이나 유지보수가 너무 힘들었다.
현실 세계의 사물 단위처럼 객체 단위로 프로그램을 구성하여 코드 모듈화와 재사용성을 극대화하기 위해 등장했다.

## 2. 클래스 vs 객체

### 클래스
무언가를 만들어내기 위한 설계도, 틀, 템플릿이다. (예: 자동차 설계도)

### 객체
클래스라는 틀을 기반으로 메모리상에 실제로 찍혀나온 결과물이다. 인스턴스라고도 부른다. (예: 도면을 보고 만든 실제 자동차)

```java
// 클래스(설계도) 정의 기본 예시
class Car {
    String color;
    int speed;
}

// 객체(실제 결과물) 생성
Car c = new Car();
```

(Blue 백엔드 작성 예시: 사용자 DTO 클래스 구조)
```java
class MemberDTO {
    String memberId;
    String name;
}
MemberDTO newMember = new MemberDTO();
newMember.memberId = "user01";
```

간결하게 정리하면 클래스는 타입이고 객체는 실제 메모리에 뜬 실체이다.

## 3. 객체의 구성 요소

객체는 크게 두 가지 성질을 갖는다.

- 변수: 상태, 속성 (예: 색깔, 현재 속도)
- 메서드: 동작, 행동 (예: 엑셀 밟기, 브레이크 밟기)

## 4. 변수 종류 (중요)

### 1) 인스턴스 변수
가장 기본이 되는 변수. 객체가 만들어질 때마다 객체별로 따로 생성되는 종속 변수다. (아반떼는 파랑, 소나타는 검정이듯 각 차마다 다름)

### 2) 클래스 변수 (static)
모든 객체가 공동으로 하나의 똑같은 값을 공유할 때 쓴다. static 키워드를 붙인다. (모든 차는 바퀴가 4개니까 값을 서로 공유함)

(NovaCinema 백엔드 작성 예시: 영화 예매 한도 고정값)
```java
// 영화 예매 한도는 어느 회원이든 기본 시스템상 5명으로 공유하니까 static
class ReservationService {
    static int MAX_BOOKING_COUNT = 5;
}
```

### 3) 지역 변수
메서드 안에서만 잠깐 살다가 끝나면 깔끔하게 지워지는 임시 변수다.

## 5. 메서드와 호출 스택

C언어 등의 함수 개념과 본질은 같지만, 특정 클래스 객체 내부에 필수로 종속되어 있다는 철학적 차이가 있다.
자바에서 내부적으로 이뤄지는 데이터 메모리 구조의 핵심은 Call Stack(호출 스택)이다.
메서드를 실행하면 박스 위에 박스를 쌓듯이 스택에 임시로 쌓여 올라간다. 제일 위에 존재하는 메서드가 끝나면 그 즉시 제거되며 자신을 불렀던 이전 층으로 알아서 되돌아간다.

## 6. 기본형 vs 참조형 매개변수 (중요)

- 기본형 (Primitive Type): 진짜 숫자(값) 자체를 변수에 저장한다. 복사본이 넘어간다.
- 참조형 (Reference Type): 객체가 존재하는 실제 번지수 주소값을 넘긴다. 함수 안쪽에서 타고 들어가서 원본 데이터를 직접 바꿀 수 있다.

## 7. 메서드 오버로딩 (Overloading)

똑같은 이름을 가진 메서드를 여러 개 만들어 두는 것이다.

조건:
- 이름은 무조건 같아야 함.
- 자바가 구별할 수 있어야 하니 매개변수 점수나 타입이 서로 달라야 함.

## 8. 생성자 (Constructor)

객체가 new 연산자를 만나 처음 탄생할 때 단 한 번 무조건 실행되는 초기화 메서드다.
- 이름이 무조건 클래스 이름과 똑같아야 한다.
- 반환 타입 자체가 애초에 없다. (void도 안 적음)

(Blue 백엔드 작성 예시: Login DTO 초기 데이터 세팅 생성자)
```java
class LoginRequest {
    String id;
    String password;

    // 객체 태어나자마자 아이디/비번 세팅
    LoginRequest(String inputId, String inputPw) {
        this.id = inputId;
        this.password = inputPw;
    }
}
LoginRequest loginData = new LoginRequest("test01", "pass1234");
```

## 9. this 와 this()

- this: 메서드 코드 안에서 자기 자신 객체의 주소를 가리킬 때 쓴다. 매개변수와 내 인스턴스 변수 이름이 겹칠 때 강제로 붙인다.
- this(): 생성자 안에서 중복 코드를 피하고 내 클래스의 또 다른 버전의 생성자를 부를 때 쓴다. 첫 줄에만 쓸 수 있다.

---

# Chapter 07. 객체지향 프로그래밍 II

객체지향다운 맛을 내는 장. 7장에서는 상속, 오버라이딩, 다형성, 캡슐화가 핵심이다.

## 1. 상속 (Inheritance)

처음부터 100% 다 짜는 게 아니라 유용한 기존 클래스의 틀을 물려받고 확장하여 새 클래스를 만드는 것이다. 부모 쪽에 공통 기능을 짜두고 자식은 그 코드를 재사용성의 이점으로 가져간다.

```java
class Child extends Parent {}
```

(NovaCinema 백엔드 작성 예시: 커스텀 에외 상속 구현)
```java
// 자바가 이미 만들어둔 RuntimeException의 수많은 기능을 모두 상속받고
// 내 입맛에 맞는 예매 전용 에러로 이름만 달아 창조함
class MovieBookingException extends RuntimeException {
}
```

자식은 부모보다 기능이 언제나 더 추가되는 형태이므로, 덩치가 더 크다. (역은 성립 안 함)

## 2. 메서드 오버라이딩 (Overriding)

부모에게 상속받은 메서드가 맘에 안 들 때, 본인 코드 블록에서 내 입맛대로 알맹이만 재정의해서 덮어쓰는 문법이다.

오버라이딩 3대 조건:
1. 이름이 같을 것
2. 매개변수가 같을 것
3. 반환 타입이 무조건 같을 것

(HulaHoop 백엔드 작성 예시: AI 인텐트 핸들러 오버라이딩)
```java
class AiIntentHandler {
    void handleIntent() {
        System.out.println("기본 AI 의도 파악");
    }
}

class MovieCancelHandler extends AiIntentHandler {
    @Override
    void handleIntent() {
        System.out.println("기본 룰 말고, 영화 취소 관련 로직으로 오버라이딩 특별 처리");
    }
}
```

## 3. super 와 super()

- super: 자식 인스턴스 안에서 오버라이딩에 가려진 진짜 오리지널 부모 측의 속성/메서드를 다시 쓸 때 명시하는 키워드.
- super(): 자식 객체가 만들어질 때 무조건 뼈대를 잡는 부모부터 먼저 태어나야 하므로, 부모 생성자를 강제로 부를 때 첫 줄에 쓴다.

## 4. 캡슐화와 접근 제어자

외부 사람이 내 클래스의 민감 변수를 직접 맘대로 손대는 것을 막기 위해, 데이터 보호막을 치는 개념이다.
데이터 무결성을 보호하는 최선봉 기술이다.

```java
private int balance; // 외부 접근 완전 차단

// 합법적으로 허가된 통로(메서드)만 열어줌
public void deposit(int money){
    if(money >= 0) this.balance += money; // 타당한 값인지 여기서 검증 후 세팅
}
```

(Blue 백엔드 작성 예시: 프론트 비밀번호 검증 방어)
```java
class Member {
    private String password;

    public void setPassword(String newPw) {
        if(newPw.length() >= 8) { // 자바스크립트는 믿을수없으니 백엔드에서 다시한번 방어
            this.password = newPw;
        }
    }
}
```

## 5. 다형성 (Polymorphism) - 초핵심 심화 부분

단순하게 말해 하나의 상위(부모) 타입 변수로 여러 종류의 하위(자식) 객체들을 통일해서 다루는 것이다.

(기본 교재 예시)
```java
Person p = new Student();
```

이걸 왜 쓰나? 구현 방식이 서로 다른 여러 자식 객체들이라도 결국 같은 부모로 묶인다면, 코드 작성 시 배열이나 매개변수 하나로 묶어 유연하게 관리할 수 있기 때문이다.

(HulaHoop 백엔드 결제 연동 작성 예시: 부모 통제 다형성)
```java
Payment[] payMethods = new Payment[3];
payMethods[0] = new KakaoPay();
payMethods[1] = new TossPay();

// 뭐가됐든 하위 결제수단들이 부모에 있는 공통 메서드만 호출해주면 오버라이딩된 각자의 결제가 나감
for(int i = 0; i < 2; i++) {
    payMethods[i].processPay(); 
}
```

### 다형성의 한계와 다운캐스팅

부모 타입(Payment)으로 형변환되어 묶였기 때문에, 자바 컴파일러는 이 변수를 부모 타입으로만 인식하여 부모 클래스에 선언된 공통 메서드만 호출할 수 있게 제약을 건다.
만약 결제 수단 중에서도 카카오페이만이 가진 고유 기능(카톡 문자 전송 메서드)을 꼭 써야 한다면, 강제로 원래의 자식 타입(KakaoPay)으로 형변환(다운캐스팅)을 해줘야 제약이 풀린다.

```java
Payment p = new KakaoPay();
// p.sendKakaoMsg(); // 에러 발생! 자바는 p를 그냥 Payment로 알고 있음.

KakaoPay kakao = (KakaoPay) p; // 강제 다운캐스팅 명시
kakao.sendKakaoMsg(); // 자기 신분을 찾았으므로 카카오 전용 기능이 허용됨
```

---

# 총 요약 마무리

- Chapter 06: 객체 하나하나를 제대로 설계하고 틀에 넣어 찍어내는 기초(변수, 메서드, 생성자)를 내 프로젝트 예시와 함께 정리함.
- Chapter 07: 객체들이 서로 부모 자식으로 맺어 돌아가는 핵심 원리(상속, 오버라이딩, 다운캐스팅)를 실무 코드와 비교하여 정리함.

이 뼈대 위에서, 곧 실무 자바 프로그램 개발 구조의 기반이 되는 인터페이스와 추상클래스로 넘어간다.

# Java Study Notes (자바의 정석 4판 10장 정리)

## 개요

자바의 정석 10장은 날짜/시간 처리와 숫자/문자 포맷팅 파트다.
API 응답 날짜 형식 통일, 결제 금액 콤마 표시, D-day 계산 등 실무에서 자주 쓰이는 기능들을 다룬다.

---

# 1. 현재 날짜와 시간 가져오기

자바 8 이후부터는 java.time 패키지의 클래스들을 쓰는 게 표준이다.

| 클래스 | 의미 |
|---|---|
| LocalDate | 날짜만 (년/월/일) |
| LocalTime | 시간만 (시/분/초) |
| LocalDateTime | 날짜 + 시간 전부 |

```java
LocalDate today = LocalDate.now();        // 2026-04-09
LocalTime now = LocalTime.now();          // 21:00:15.123
LocalDateTime dateTime = LocalDateTime.now(); // 2026-04-09T21:00:15.123
```

(Blue 백엔드 작성 예시: 예약 생성 시 현재 시간 기록)
```java
LocalDateTime reservedAt = LocalDateTime.now();
reservation.setReservedAt(reservedAt);
```

---

# 2. 날짜 계산 (더하기, 빼기)

날짜 객체는 불변이라 더하거나 빼면 새 객체가 반환된다. 기존 객체는 그대로다.

```java
LocalDate today = LocalDate.now();

today.plusDays(10);    // 10일 후
today.plusMonths(1);   // 1달 후
today.plusYears(1);    // 1년 후
today.minusDays(3);    // 3일 전
```

(Blue 백엔드 작성 예시: 무료 체험 기간 만료일 계산)
```java
LocalDate joinDate = LocalDate.now();
LocalDate freeTrialEnd = joinDate.plusDays(30);
member.setFreeTrialEnd(freeTrialEnd);
```

---

# 3. 날짜 비교

```java
LocalDate d1 = LocalDate.of(2026, 4, 1);
LocalDate d2 = LocalDate.of(2026, 4, 9);

d1.isBefore(d2); // true
d1.isAfter(d2);  // false
d1.isEqual(d2);  // false
```

(Blue 백엔드 작성 예시: 예약 날짜 유효성 검증)
```java
if (requestedDate.isBefore(LocalDate.now())) {
    throw new BusinessException("이미 지난 날짜로는 예약이 불가합니다.");
}
```

---

# 4. 두 날짜 사이 차이 계산

ChronoUnit을 이용하면 두 날짜 사이의 차이를 일/월/년 단위로 계산할 수 있다.

```java
import java.time.temporal.ChronoUnit;

long days = ChronoUnit.DAYS.between(d1, d2);   // 8
long months = ChronoUnit.MONTHS.between(d1, d2); // 0
```

(Blue 백엔드 작성 예시: 구독 남은 기간 표시)
```java
long daysLeft = ChronoUnit.DAYS.between(LocalDate.now(), freeTrialEnd);
response.setDaysLeft(daysLeft);
```

---

# 5. 날짜 포맷 변환 (DateTimeFormatter)

자바 기본 출력 형식을 원하는 형태로 바꿀 때 쓴다.

패턴 규칙:

| 패턴 | 의미 |
|---|---|
| yyyy | 년 (4자리) |
| MM | 월 (2자리) |
| dd | 일 (2자리) |
| HH | 시 (24시간) |
| mm | 분 |
| ss | 초 |

```java
LocalDateTime now = LocalDateTime.now();

DateTimeFormatter f1 = DateTimeFormatter.ofPattern("yyyy년 MM월 dd일");
String koreanDate = now.format(f1); // "2026년 04월 09일"

DateTimeFormatter f2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String logDate = now.format(f2); // "2026-04-09 21:00:15"
```

(Blue 백엔드 작성 예시: API 응답 날짜 통일 포맷)
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
String formattedAt = reservation.getReservedAt().format(formatter);
response.setReservedAt(formattedAt);
```

---

# 6. 문자열 -> 날짜 객체 변환 (parse)

프론트에서 문자열로 날짜를 받았을 때, 날짜 계산을 하려면 LocalDate 객체로 파싱해야 한다.

```java
String input = "2026-04-09";
LocalDate date = LocalDate.parse(input); // "yyyy-MM-dd" 기본 지원

// 커스텀 형식
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy년 MM월 dd일");
LocalDate date2 = LocalDate.parse("2026년 04월 09일", formatter);
```

(Blue 백엔드 작성 예시: 예약 날짜 문자열 파싱)
```java
LocalDate reservationDate = LocalDate.parse(request.getReservationDate());
if (reservationDate.isBefore(LocalDate.now())) {
    throw new BusinessException("과거 날짜는 예약할 수 없습니다.");
}
```

---

# 7. 숫자 포맷팅 (NumberFormat)

숫자에 콤마 찍기, 통화 기호 붙이기 등 사용자 친화적 형태로 변환한다.

```java
import java.text.NumberFormat;

NumberFormat nf = NumberFormat.getInstance();
System.out.println(nf.format(1000000)); // "1,000,000"

NumberFormat money = NumberFormat.getCurrencyInstance();
System.out.println(money.format(10000)); // "₩10,000"
```

(Blue 백엔드 작성 예시: 결제 금액 콤마 포맷 응답)
```java
NumberFormat nf = NumberFormat.getInstance();
String formattedAmount = nf.format(payment.getAmount()) + "원";
response.setFormattedAmount(formattedAmount); // "15,000원"
```

---

# 최종 정리

- LocalDate / LocalTime / LocalDateTime: java.time 표준. 현재 날짜와 시간 처리.
- plusDays / minusDays: 날짜 더하기 빼기. 반환값이 새 객체임에 주의.
- ChronoUnit.DAYS.between(): 두 날짜 간 일 차이 계산. D-day, 구독 기간에 필수.
- DateTimeFormatter.ofPattern(): 날짜를 원하는 문자열 형식으로 변환.
- LocalDate.parse(): 문자열을 날짜 객체로 역변환.
- NumberFormat: 숫자 콤마 포맷, 통화 단위 표시.

다음 학습 노트(Java_Study_Notes_11.md)에서 컬렉션 프레임워크(List, Set, Map)로 이어진다.

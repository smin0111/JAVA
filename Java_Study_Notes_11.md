# Java Study Notes (자바의 정석 4판 11장 정리)

## 개요

자바의 정석 11장은 컬렉션 프레임워크 파트다.
백엔드 개발에서 다루는 데이터의 70%는 컬렉션으로 처리된다. DB 조회 결과, API 응답 목록, 회원 리스트 등 거의 모든 데이터 덩어리가 컬렉션 객체로 돌아다닌다.

---

# 1. 컬렉션이란

지금까지는 변수 하나씩 따로 만들었다.

```java
int a = 10;
int b = 20;
```

하지만 회원 수천 명, 영화 수백 개를 다룰 때 변수 하나씩은 불가능하다.
이를 해결하는 것이 컬렉션(Collection)이다.

자바 컬렉션 핵심 3가지:

| 종류 | 특징 | 대표 클래스 |
|---|---|---|
| List | 순서 있음, 중복 허용 | ArrayList |
| Set | 중복 없음 | HashSet |
| Map | key-value 쌍 저장 | HashMap |

---

# 2. List - ArrayList (가장 많이 씀)

순서가 있고 중복을 허용한다. 배열과 비슷하지만 크기가 동적으로 늘어난다.
실무에서는 거의 대부분 ArrayList를 쓴다.

```java
import java.util.ArrayList; // ArrayList를 쓰려면 반드시 import 해야 한다

ArrayList<String> list = new ArrayList<>();
// <>안에 String을 적어서 "이 리스트에는 문자열만 들어갈 수 있다"고 타입을 제한한다
// new ArrayList<>()로 실제 리스트 객체를 메모리에 생성한다

list.add("사과");   // 리스트 맨 뒤에 "사과"를 추가. [사과]
list.add("바나나"); // 리스트 맨 뒤에 "바나나"를 추가. [사과, 바나나]
list.add("딸기");   // 리스트 맨 뒤에 "딸기"를 추가. [사과, 바나나, 딸기]

System.out.println(list.get(0)); // get(인덱스)로 해당 위치 값을 꺼낸다. 0번 = "사과"
System.out.println(list.get(1)); // 1번 인덱스 = "바나나"

System.out.println(list.size()); // 현재 리스트에 들어있는 요소 개수를 반환. 결과: 3

list.remove(0);        // 인덱스 0번 위치("사과")를 삭제. 나머지가 앞으로 당겨짐
list.remove("바나나"); // 값("바나나")으로 직접 찾아서 삭제. 없으면 아무 일도 안 일어남

list.contains("딸기"); // 리스트 안에 "딸기"가 있는지 true/false로 반환
```

for-each 반복 (실무에서 매일 씀):
```java
for (String fruit : list) {
    // list 안의 요소를 하나씩 꺼내서 fruit 변수에 담고 블록을 실행한다
    // 배열처럼 인덱스를 쓰지 않아도 되어서 훨씬 깔끔하다
    System.out.println(fruit);
}
```

(Blue 백엔드 작성 예시: 예약 목록 응답)
```java
List<ReservationDTO> reservations = reservationMapper.findByMemberId(memberId);
// reservationMapper.findByMemberId()가 SQL 결과를 List 형태로 반환한다
// MyBatis가 각 행(row)을 ReservationDTO 객체로 변환해서 담아준다

return ResponseEntity.ok(reservations);
// List째로 그대로 ok()에 넘기면 Jackson이 JSON 배열로 직렬화해서 프론트에 내려준다
```

---

# 3. Set - HashSet (중복 제거)

중복을 자동으로 제거해주는 컬렉션이다. 순서는 보장되지 않는다.

```java
import java.util.HashSet; // HashSet 사용을 위한 import

HashSet<String> set = new HashSet<>();
// <>안에 String을 적어서 문자열만 허용하는 Set을 생성한다

set.add("사과");   // Set에 "사과" 추가. 현재: [사과]
set.add("사과");   // "사과"는 이미 있으므로 무시됨. Set은 같은 값을 두 번 허용하지 않는다
set.add("바나나"); // "바나나" 추가. 현재: [사과, 바나나] (순서는 내부 해시 기준이라 바뀔 수 있음)

System.out.println(set); // [사과, 바나나] - 중복이 제거된 결과만 남는다
System.out.println(set.size()); // 2 - 실제 저장된 고유 요소 수
set.contains("사과"); // true - Set 안에 해당 값이 있는지 확인
```

(Blue 백엔드 작성 예시: 중복 좌석 선택 방어)
```java
HashSet<String> seatSet = new HashSet<>(Arrays.asList(selectedSeats));
// Arrays.asList()로 배열을 List로 변환한 다음
// HashSet 생성자에 넘기면 List의 요소들이 자동으로 Set에 담기면서 중복이 제거된다

if (seatSet.size() != selectedSeats.length) {
    // 원래 배열 길이와 중복 제거된 Set의 크기를 비교한다
    // 크기가 다르다 = 중복이 있었다는 뜻이므로 예외를 던진다
    throw new BusinessException("중복된 좌석 번호가 포함되어 있습니다.");
}
```

---

# 4. Map - HashMap (key로 값 찾기)

key-value 쌍으로 데이터를 저장하고, key를 이용해 빠르게 값을 찾는다.
인덱스가 아닌 의미 있는 이름(key)으로 직접 꺼낼 수 있어 실무에서 매우 자주 쓰인다.

```java
import java.util.HashMap; // HashMap 사용을 위한 import

HashMap<String, String> map = new HashMap<>();
// <String, String>에서 앞이 key 타입, 뒤가 value 타입이다
// 즉 "문자열 key로 문자열 value를 찾는 Map"이라는 의미다

map.put("id1", "철수"); // key "id1"에 value "철수"를 저장
map.put("id2", "영희"); // key "id2"에 value "영희"를 저장
// 이미 있는 key에 put하면 기존 value가 새 value로 덮어씌워진다

System.out.println(map.get("id1")); // key "id1"로 value를 꺼냄. 출력: "철수"
System.out.println(map.get("없는키")); // 없는 key는 null 반환. NullPointerException 주의

map.containsKey("id1");    // key "id1"이 Map 안에 있는지 true/false
map.containsValue("영희"); // value "영희"가 Map 안에 있는지 true/false

map.remove("id2"); // key "id2"와 해당 value를 통째로 삭제
map.size();        // 현재 저장된 key-value 쌍의 개수 반환
```

Map 전체 순회:
```java
for (Map.Entry<String, String> entry : map.entrySet()) {
    // entrySet()이 Map 안의 모든 key-value 쌍을 Set 형태로 반환한다
    // 각 쌍이 Map.Entry 객체로 묶여져 있어서 entry 변수로 하나씩 받는다
    entry.getKey();   // 현재 entry의 key를 꺼낸다
    entry.getValue(); // 현재 entry의 value를 꺼낸다
    System.out.println(entry.getKey() + " : " + entry.getValue());
}
```

(Blue 백엔드 작성 예시: 토스 결제 API 응답 파싱)
```java
Map<String, Object> tossResponse = restTemplate.getForObject(url, Map.class);
// restTemplate이 토스 서버에 GET 요청을 날리고 JSON 응답을 Map으로 변환해서 받는다
// <String, Object>로 받는 이유는 value가 String, Long, Integer 등 여러 타입이 섞여서 오기 때문

String paymentKey = (String) tossResponse.get("paymentKey");
// key "paymentKey"로 value를 꺼낸다
// Map<String, Object>의 value는 Object 타입이라 (String)으로 명시적 형변환을 해야 쓸 수 있다

Long amount = ((Number) tossResponse.get("totalAmount")).longValue();
// JSON 숫자는 Integer나 Long으로 들어올 수 있어서 일단 Number로 받고
// longValue()로 Long 타입으로 변환한다. Long이나 Integer가 모두 Number를 상속하기 때문에 가능하다
```

---

# 5. Iterator (반복기)

컬렉션을 순회하는 자바 공식 방법이다.
for-each로 순회 중 삭제하면 ConcurrentModificationException 에러가 터진다.
Iterator의 remove()를 써야 반복 중 삭제가 안전하게 가능하다.

```java
Iterator<String> it = list.iterator();
// list.iterator()가 이 리스트를 순서대로 훑을 수 있는 Iterator 객체를 반환한다
// Iterator<String>으로 선언해서 꺼낼 때마다 String 타입으로 받을 수 있게 한다

while (it.hasNext()) {
    // hasNext()는 아직 꺼내지 않은 다음 요소가 있으면 true, 없으면 false를 반환한다
    // 이 조건이 false가 되는 순간 while 루프가 끝난다

    String s = it.next();
    // next()로 다음 요소를 꺼내서 s에 담는다
    // 호출할 때마다 내부 커서가 한 칸씩 앞으로 이동한다

    if (s.equals("사과")) {
        it.remove();
        // Iterator 안에 있는 remove()를 써야 반복 도중 삭제가 안전하다
        // list.remove()를 직접 쓰면 구조가 바뀌어서 ConcurrentModificationException이 터진다
    }
}
```

(Blue 백엔드 작성 예시: 만료된 예약 목록 정리)
```java
Iterator<ReservationDTO> it = reservations.iterator();
// reservations 리스트를 순서대로 훑는 Iterator를 꺼낸다

while (it.hasNext()) {
    ReservationDTO r = it.next();
    // 다음 예약 DTO 객체를 꺼낸다

    if (r.getExpiredAt().isBefore(LocalDate.now())) {
        // 해당 예약의 만료일이 오늘보다 이전(과거)이면 만료된 예약으로 판단
        it.remove();
        // Iterator를 통해 해당 요소를 리스트에서 안전하게 제거한다
    }
}
```

---

# 6. Collections 유틸 클래스

컬렉션 전용 유틸 메서드 모음이다. static 메서드라 new 없이 바로 호출 가능하다.

```java
import java.util.Collections; // Collections 유틸 클래스 import

Collections.sort(list);
// list를 오름차순으로 정렬한다. 문자열이면 사전순, 숫자면 작은 순서로 정렬된다
// 내부적으로 Comparable의 compareTo()를 호출하므로 커스텀 객체는 Comparable 구현이 필요하다

Collections.reverse(list);
// list의 요소 순서를 뒤집는다. 정렬 후 reverse()하면 내림차순 효과

Collections.shuffle(list);
// list의 요소 순서를 랜덤으로 섞는다. 매번 실행마다 순서가 달라진다

Collections.max(list);
// list에서 가장 큰 값을 반환한다 (Comparable 기준으로 비교)

Collections.min(list);
// list에서 가장 작은 값을 반환한다
```

---

# 7. Comparable / Comparator (정렬 기준 직접 만들기)

문자열이나 숫자는 Collections.sort() 로 바로 정렬된다.
하지만 내가 만든 객체(예: ReservationDTO)는 정렬 기준이 없어서 직접 만들어야 한다.

## Comparable: 클래스 자체에 기본 정렬 기준을 심는 방법

```java
class Reservation implements Comparable<Reservation> {
    // Comparable<Reservation>을 implements하겠다고 선언해야 compareTo 메서드를 오버라이딩할 수 있다
    // "이 클래스의 객체들끼리 크기를 비교할 수 있다"는 계약서에 사인하는 것

    LocalDate date;

    @Override
    public int compareTo(Reservation other) {
        // 나(this)와 상대(other)를 비교하는 메서드다
        // Collections.sort() 호출 시 자바가 내부적으로 이 메서드를 호출해서 순서를 정한다

        return this.date.compareTo(other.date);
        // 반환값이 음수면 this가 other보다 앞에 온다 (오름차순)
        // 반환값이 양수면 this가 other보다 뒤에 온다
        // 반환값이 0이면 같은 순위로 본다
        // LocalDate의 compareTo는 날짜가 이를수록 음수를 반환하므로 날짜 오름차순 정렬이 된다
    }
}

Collections.sort(reservations);
// Comparable이 구현되어 있으므로 compareTo를 자동으로 사용해서 날짜 오름차순으로 정렬된다
```

## Comparator: 외부에서 즉석으로 정렬 기준을 만드는 방법

클래스를 바꾸지 않고, 정렬 시점에 기준을 람다로 바로 넣을 때 쓴다.

```java
reservations.sort((a, b) -> b.getDate().compareTo(a.getDate()));
// sort() 안에 람다식으로 Comparator를 즉석 정의한다
// (a, b)는 비교할 두 Reservation 객체다
// b.getDate().compareTo(a.getDate())는 b의 날짜를 기준으로 a와 비교하는 것
// b가 a보다 최신(큰 날짜)이면 음수가 나와서 b가 앞에 오는 역순(최신순) 정렬이 된다
```

(Blue 백엔드 작성 예시: 예약 내역 최신순 정렬)
```java
List<ReservationDTO> reservations = reservationMapper.findByMemberId(memberId);
// MyBatis Mapper를 통해 해당 회원의 예약 목록을 전체 조회한다

reservations.sort((a, b) -> b.getReservedAt().compareTo(a.getReservedAt()));
// 예약 시각(reservedAt) 기준으로 b와 a를 비교해서 최신 예약이 리스트 앞쪽에 오도록 내림차순 정렬한다
// SQL에서 ORDER BY로 처리해도 되지만 자바 레이어에서 처리하는 방식이다

return ResponseEntity.ok(reservations);
// 정렬된 List를 그대로 200 OK 응답에 담아서 프론트에 내려준다
```

---

# 8. HashSet / HashMap이 중복을 판단하는 원리

Set과 Map이 중복을 거를 수 있는 이유가 바로 9장에서 배운 equals()와 hashCode()다.
자바는 두 객체가 같은지 판단할 때 hashCode()가 같고, equals()도 true인 경우에만 "같다"고 본다.
그래서 커스텀 객체를 Set/Map 키로 쓸 때는 두 메서드를 반드시 오버라이딩해야 중복 제거가 제대로 동작한다.

순서:
1. 새 요소가 들어오면 먼저 hashCode()를 실행해서 해시값으로 버킷(저장 공간) 위치를 정한다.
2. 같은 버킷 안에 이미 요소가 있으면 equals()로 실제 내용이 같은지 한 번 더 비교한다.
3. hashCode도 같고 equals도 true이면 완전히 같은 객체로 보고 중복 처리(무시)한다.

---

# 최종 정리

- ArrayList: 순서 있고 중복 허용하는 목록. DB 조회 결과 담을 때 필수.
- HashSet: 중복 자동 제거. 중복 방지 로직(좌석, 좋아요 등)에 활용.
- HashMap: key-value 저장. API 응답 파싱, key로 빠른 값 조회에 활용.
- Iterator: 반복 중 삭제가 필요할 때 for-each 대신 사용.
- Collections: sort, shuffle 등 컬렉션 유틸 메서드 모음.
- Comparable / Comparator: 커스텀 객체 정렬 기준 정의. 최신순, 가격순 정렬 등에 필수.
- equals / hashCode: Set/Map의 중복 판단 원리. 커스텀 객체 키로 쓸 때 반드시 오버라이딩.

# Java Study Notes (자바의 정석 4판 14장 정리 - 람다와 스트림)

---

# 14장 람다와 스트림 (Lambda & Stream)

여기부터가 현대 자바의 핵심 기술이다.
실무 최적화, 코딩 테스트, Spring 프레임워크 전반에 연결되는 파트다.

한 줄 요약:
코드를 함수처럼 다루는 방법(람다)과 데이터를 선언형으로 처리하는 방법(스트림)의 결합이다.

---

# 파트 1. 람다식 (Lambda Expression)

# 1. 람다식 등장 배경

옛날 자바에는 함수 개념 자체가 없었다. 메서드 하나만 필요해도 익명 클래스를 통째로 만들어야 했다.

옛날 방식:
```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
}).start();
```

람다 도입 후 (자바 8):
```java
new Thread(() -> System.out.println("Hello")).start();
```

---

# 2. 람다식 기본 문법

기존 메서드:
```java
int add(int a, int b) {
    return a + b;
}
```

람다 변환:
```java
(a, b) -> a + b
```

구조: (매개변수) -> { 실행 로직 }

---

# 3. 람다 생략 규칙 (시험 단골)

```java
(int a, int b) -> a + b   // 기본
(a, b) -> a + b           // 타입 생략 가능
x -> x * x                // 매개변수 1개면 소괄호 생략
(a, b) -> a + b           // 실행문 1줄이면 중괄호 생략
(a, b) -> a + b           // return도 자동 생략
```

---

# 4. 함수형 인터페이스

람다는 이름이 없어서 담을 타입이 필요하다. 추상 메서드가 딱 1개인 인터페이스가 그 역할을 한다.

```java
@FunctionalInterface
interface MyFunction {
    int max(int a, int b);
}

MyFunction f = (a, b) -> a > b ? a : b;
System.out.println(f.max(3, 5)); // 5
```

@FunctionalInterface: 추상 메서드가 2개 이상이면 컴파일 에러로 실수를 방지한다.

---

# 5. 자바 내장 함수형 인터페이스 4개 (핵심 암기)

| 인터페이스 | 역할 | 메서드 |
|---|---|---|
| Consumer | 소비 (결과 없음) | void accept(T t) |
| Supplier | 공급 (입력 없음) | T get() |
| Function | T -> R 변환 | R apply(T t) |
| Predicate | 조건 판별 | boolean test(T t) |

Consumer:
```java
Consumer<String> c = s -> System.out.println(s);
c.accept("Hello");
```

Supplier:
```java
Supplier<Integer> s = () -> (int)(Math.random() * 100);
System.out.println(s.get());
```

Function:
```java
Function<String, Integer> f = s -> s.length();
System.out.println(f.apply("Hello")); // 5
```

Predicate (스트림 filter에서 핵심으로 쓰임):
```java
Predicate<Integer> p = x -> x > 0;
System.out.println(p.test(5)); // true

// Blue 백엔드 예시: 활성 상태 회원만 필터링할 때 사용한다
// memberList.stream().filter(m -> m.isActive()).collect(Collectors.toList());
```

---

# 6. 컬렉션과 람다 (forEach)

```java
// 기존 for-each
for(String s : list) {
    System.out.println(s);
}

// 람다 방식 (forEach 내부가 Consumer를 사용한다)
list.forEach(s -> System.out.println(s));
```

---

# 7. 메서드 참조

람다가 너무 뻔할 때 더 압축하는 방법이다.

```java
// 기존 람다
list.forEach(s -> System.out.println(s));

// 메서드 참조로 압축 (클래스::메서드)
list.forEach(System.out::println);
```

생성자 참조:
```java
Supplier<String> s = String::new; // () -> new String() 과 동일
```

---
---

# 파트 2. 스트림 (Stream)

# 1. 스트림 탄생 이유

짝수만 골라 정렬 후 출력하려면?

옛날 방식:
```java
List<Integer> result = new ArrayList<>();
for(Integer i : list){
    if(i % 2 == 0){ result.add(i); }
}
Collections.sort(result);
for(Integer i : result){ System.out.println(i); }
```

코드가 길고, 가독성이 낮고, 병렬 처리가 어렵다. 그래서 스트림이 등장했다.

---

# 2. 스트림 기본 구조 (시험 필수)

스트림은 항상 3단계다:
1. 스트림 생성
2. 중간 연산 (여러 개 연결 가능)
3. 최종 연산 (1번만, 실행 시 스트림 소멸)

```java
list.stream()                        // 1. 스트림 생성
    .filter(x -> x % 2 == 0)        // 2. 중간 연산: 짝수만 통과
    .sorted()                        // 2. 중간 연산: 정렬
    .forEach(System.out::println);  // 3. 최종 연산: 출력 후 소멸
```

---

# 3. 스트림 생성 (1단계)

```java
Stream<Integer> s = list.stream();           // 컬렉션
Stream<String> s = Arrays.stream(arr);       // 배열
Stream<Integer> s = Stream.of(1, 2, 3, 4);  // 값 직접 지정
```

---

# 4. 중간 연산 (2단계)

최종 연산이 호출되기 전까지는 실행되지 않는다 (Lazy 지연 연산).

| 메서드 | 역할 |
|---|---|
| filter | 조건으로 걸러냄 |
| map | 값을 다른 형태로 변환 |
| sorted | 정렬 |
| distinct | 중복 제거 |
| limit | 개수 제한 |

filter:
```java
list.stream()
    .filter(x -> x > 10); // 10보다 큰 값만 통과
```

map:
```java
list.stream()
    .map(x -> x * 2);        // 각 숫자를 2배로 변환

list.stream()
    .map(String::length);     // 문자열을 길이로 변환
```

Blue 백엔드 예시 (map 실무 활용):
```java
// Member 엔티티 리스트를 MemberDTO 리스트로 변환할 때 사용한다
List<MemberDTO> dtoList = memberList.stream()
    .map(m -> new MemberDTO(m.getId(), m.getName()))
    .collect(Collectors.toList());
```

sorted:
```java
stream.sorted();               // 오름차순
stream.sorted((a, b) -> b-a); // 내림차순
```

---

# 5. 최종 연산 (3단계)

최종 연산이 호출되어야만 중간 연산들이 실제로 작동한다.

forEach:
```java
stream.forEach(System.out::println);
```

count:
```java
long cnt = stream.count();
```

collect (가장 중요):
```java
// 필터링된 결과를 새 List에 담아 반환한다
List<Integer> result = list.stream()
    .filter(x -> x % 2 == 0)
    .collect(Collectors.toList());

// 자주 쓰는 형태: toList(), toSet(), toMap()
```

reduce (누적 연산):
```java
// 초기값 0에서 시작, 모든 요소를 계속 더해나가는 누산기
int sum = list.stream()
    .reduce(0, (a, b) -> a + b);
```

---

# 6. 스트림 핵심 특징 3가지 (시험 단골)

1. 원본 데이터를 변경하지 않는다.
```java
list.stream().filter(...); // list 원본은 그대로, 새 결과만 만들어낸다
```

2. 일회용이다 (재사용 불가).
```java
Stream s = list.stream();
s.count();      // 이 시점에 스트림 소멸
s.forEach(...); // 에러 발생 (IllegalStateException)
```

3. 지연 연산 (Lazy): 중간 연산은 최종 연산이 호출될 때까지 실행되지 않는다.

---

# 7. 병렬 스트림

CPU 멀티코어를 자동 활용해 병렬 처리한다.

```java
list.parallelStream()
    .forEach(System.out::println);
```

장점: 대용량 데이터 처리 속도가 빨라진다.
단점: 출력 순서 보장이 안된다. 작은 데이터에는 오히려 느려진다.

---

# 8. 스트림 완성형 예시

"짝수만 골라 제곱한 뒤 새 리스트로 반환"

```java
List<Integer> result = list.stream()
    .filter(x -> x % 2 == 0)       // 짝수만 통과
    .map(x -> x * x)               // 제곱으로 변환
    .collect(Collectors.toList()); // 새 리스트에 담아 반환
```

---

# 14장 최종 요약

람다:
- 익명 클래스를 화살표 문법으로 간결하게 줄인 것이다
- 반드시 함수형 인터페이스 타입에 담아야 한다
- Consumer(소비) / Supplier(공급) / Function(변환) / Predicate(조건) 4개 반드시 암기

스트림:
- 구조: 생성 -> 중간 연산(filter, map, sorted) -> 최종 연산(collect, forEach, reduce)
- 원본 변경 없음, 일회용, 최종 연산 전까지 지연 실행(Lazy)
- parallelStream()으로 간단하게 병렬 처리 가능

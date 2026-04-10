# Java Study Notes (자바의 정석 4판 12장 정리)

## 개요

자바의 정석 12장은 제네릭스(Generics) 파트다.
제네릭스는 클래스나 메서드를 만들 때 타입을 하나로 고정하지 않고 파라미터처럼 외부에서 지정하게 해주는 기능이다. 실무에서는 공통 응답 포맷(ApiResponse<T>)을 설계하거나, List/Map 같은 컬렉션을 안전하게 다룰 때 무조건 쓰게 된다.

---

# 1. 제네릭스란?

옛날 방식 (제네릭스 도입 이전):
```java
ArrayList list = new ArrayList();
list.add("hello");
list.add(10); // 아무 타입이나 막 들어간다 (문제점 발생 시작점)

// 리스트에서 꺼낼 때 기본적으로 Object로 나오기 때문에 원래 타입으로 일일이 형변환해야 한다
String s = (String) list.get(0);

// 만약 실수로 10이 들어있는 인덱스를 String으로 바꾸려 하면 런타임 에러(ClassCastException)가 터진다
```

제네릭스 도입 후:
```java
ArrayList<String> list = new ArrayList<>();
// <> 안에 String을 적어서, "이 리스트에는 문자열만 넣을 수 있다"고 엄격하게 제한한다

list.add("hello"); // 통과

// list.add(10);
// 문자열이 아니므로 컴파일러가 에러를 띄워 미리 방어할 수 있다

String s = list.get(0);
// 이미 내부가 String임을 자바가 알고 있으므로, 형변환 없이 바로 꺼낼 수 있다
```

(Blue 백엔드 작성 예시: API 공통 응답 래퍼 구현)
```java
public class ApiResponse<T> {
    // T라는 타입 변수를 써서 들어올 데이터의 타입을 동적으로 받는다
    private int status;
    private String message;
    private T data; // 객체가 생성되는 시점에 T가 멤버, 예약 등 구체적인 클래스로 결정됨

    public ApiResponse(int status, String message, T data) {
        this.status = status;
        this.message = message;
        this.data = data;
    }
}

// Controller에서 사용할 때
ApiResponse<MemberDTO> response = new ApiResponse<>(200, "성공", memberDto);
// T가 MemberDTO로 치환되어 데이터가 안전하게 담기고 프론트로 전달된다
```

---

# 2. 제네릭의 장점 (핵심 강점)

1. 타입 안정성 제공: 의도하지 않은 타입의 데이터가 섞여 들어가는 걸 컴파일(코드 작성) 단계에서 막아준다.
2. 형변환 제거: 저장된 객체를 꺼낼 때 원래 타입으로 자동 변환되므로 코드가 훨씬 깔끔해진다.
3. 코드 재사용: 여러 타입의 데이터를 동일한 로직으로 처리할 수 있는 공통 클래스/메서드를 만들 수 있다.

---

# 3. 타입 변수와 용어 정리

제네릭 클래스를 선언할 때 쓰는 `<T>`를 타입 변수라고 부른다.

주로 쓰는 문자 규칙 (알파벳 아무거나 써도 되지만 암묵적 룰이 있다):
- T: Type (타입)
- E: Element (요소, List<E>처럼 컨테이너 안의 내용물)
- K: Key (키, Map<K, V>에서)
- V: Value (값)
- N: Number (숫자)

용어 정리 (시험/면접 단골):
```java
Box<String> box = new Box<>();
```
- `Box<T>` : 제네릭 클래스
- `T`      : 타입 변수
- `Box<String>` : 제네릭 타입 호출 (매개변수화된 타입)
- `String` : 타입 인자 (대입된 타입)

---

# 4. 제네릭 클래스 만들기

클래스 이름 옆에 `<T>`를 붙여서 선언한다.

```java
class Box<T> {
    // 클래스 내부에서는 T를 마치 존재하는 클래스처럼 가져다 쓴다
    T item;

    void setItem(T item) {
        this.item = item;
    }

    T getItem() {
        return item;
    }
}

// 사용할 때
Box<String> box = new Box<>();
// 객체가 생성되는 이 타이밍에 클래스 내부의 T가 전부 String으로 변신한다
box.setItem("Hello");
```

다이아몬드 연산자 `<>`
자바 7부터는 코드를 줄이기 위해, 생성자 쪽의 타입 인자를 생략할 수 있게 됐다. 컴파일러가 왼쪽 변수의 타입을 보고 똑같이 유추해서 채워준다.
```java
Box<String> box1 = new Box<String>(); // 예전 방식 (입력하기 김)
Box<String> box2 = new Box<>();       // 최신 방식 (다이아몬드 연산자로 생략 가능)
```

---

# 5. 제네릭 메서드

클래스 전체를 통째로 제네릭으로 하지 않고, 특정 메서드 하나만 제네릭으로 만들 수도 있다.
이때 선언은 반환 타입 바로 앞에 `<T>`를 붙인다.

```java
static <T> void print(T t) {
    // 메서드가 호출될 때 넘겨받는 매개변수 타입에 따라서 T가 그때그때 결정된다
    System.out.println(t);
}

// 사용할 때
print("hello"); // 넘겨준 게 문자열이니 T는 String
print(10);      // 넘겨준 게 숫자니 T는 Integer
```

---

# 6. 제한된 제네릭 (extends)

`<T>`는 기본적으로 세상의 모든 객체를 다 받을 수 있다.
그런데 어떤 로직은 특정 타입이나 그 관련 부모/자식들만 받아야 고장나지 않는데, 이럴 때 `extends` 키워드로 들어올 타입을 제한한다.

```java
class Box<T extends Number> {
    // T로는 무조건 Number 클래스의 자식들(Integer, Double, Long 등)만 들어올 수 있다
}

Box<Integer> box1 = new Box<>(); // Ok (Integer는 Number의 자식)
// Box<String> box2 = new Box<>(); // 컴파일 에러 발생 (String은 핏줄이 다름)
```

인터페이스로 제한할 때의 특징:
인터페이스를 강제할 때도 `implements` 대신 똑같이 `extends`를 쓴다. 여러 개를 제한할 때는 `&` 기호로 묶는다.
```java
class Box<T extends Number & Comparable> {
    // Number의 자식이면서, Comparable 인터페이스까지 함께 구현한 타입만 들어올 수 있다
}
```

---

# 7. 와일드카드 (?) (가장 어려운 핵심 파트)

제네릭은 상속 관계가 있더라도 타입이 다르면 아예 남남으로 본다. (예: `List<Object>` 타입 변수에 `List<String>` 객체를 넣을 수 없음)
안전을 위해 빡빡하게 검사하는 건 좋은데, 가끔 유연함이 필요할 때 사용하는 것이 바로 와일드카드 `?` 이다. "무엇이든"이라는 뜻이다.

| 문법 | 의미 |
|---|---|
| `<?>` | 제한 없음 (아무 타입이나 다 됨) |
| `<? extends T>` | 상한 제한 (T 거나 T의 자식들만 가능) |
| `<? super T>`   | 하한 제한 (T 거나 T의 부모, 조상들만 가능) |

```java
// 1. <? extends T> 상한 제한
List<? extends Number> list1;
// Number를 상속받은 Integer, Double 리스트 등을 통째로 던져도 안전하게 받을 수 있다
// 대신 안에서 꺼내도 Number보다 높은 타입으로는 못 나온다는 안전 장치가 걸려있다

// 2. <? super T> 하한 제한
List<? super Integer> list2;
// Integer 묶음이거나 그 부모인 Number 또는 Object 묶음만 들어올 수 있다
```

(Blue 백엔드 작성 예시: 숫자를 다루는 서비스 메서드의 파라미터)
```java
// 어떤 컬렉션이든 숫자가 담겨 있다면 평균을 구하는 공통 메서드
public double calculateAverage(List<? extends Number> numbers) {
    // List 안의 타입이 Integer든 Double이든 Number 자식이기만 하면 다 받아준다
    double sum = 0.0;
    for (Number num : numbers) {
        sum += num.doubleValue();
    }
    return sum / numbers.size();
}
```

---

# 8. 제네릭 사용 금지 구역 (주의할 점)

이건 면접이나 객관식 시험에 자주 나오는 내용이다.

1. 제네릭 타입의 배열 생성 불가:
객체도 안 만들어졌는데 배열 크기를 미리 잡는 등의 런타임 타입 사고를 막기 위해 자바 컴파일러 선에서 차단해 놨다.
```java
// T[] arr = new T[10]; // 컴파일 에러
ArrayList<T> list = new ArrayList<>(); // 대신 ArrayList를 써야 한다
```

2. static 변수에 T 사용 불가:
static 변수는 프로그램이 시작될 때 (객체를 만들기도 전에) 미리 메모리에 올라가 버린다.
하지만 T는 나중에 누군가가 `new Box<String>()` 으로 객체를 생성할 때 비로소 결정되는 값이므로 타이밍이 맞지 않아 쓸 수 없다.
```java
class Box<T> {
    // static T item; // 여기서 T가 뭔지 static 시점에는 알 수 없으므로 에러
}
```

3. 제네릭끼리 억지 형변환 금지:
```java
Box<String> b1 = new Box<String>();
// Box<Object> b2 = (Box<Object>) b1; 
// 이런 억지 형변환은 안 된다. 이런 유연함이 필요할 땐 Box<?> 같은 와일드카드를 써야 한다
```

---

# 9. 12장 핵심 요약

- 제네릭스 목표: 컴파일 타임에 타입 체크를 철저히 해서 안정성을 높이고, 개발자가 매번 형변환할 귀찮음을 없애기 위함이다.
- `<T>`: 변할 수 있는 타입 변수 기호. 다이아몬드 연산자 `<>`로 생성 시 생략 가능.
- `<T extends 클래스>`: 들어올 수 있는 클래스의 부모 종류를 한정해서 상한선 제한.
- `<?>`, `<? extends T>`, `<? super T>`: 빡빡한 제네릭 검사를 유연하게 통과시키기 위한 와일드카드 편법.
- 불가능한 것: T타입 배열 만들기 금지, static 변수에 T 사용 금지.

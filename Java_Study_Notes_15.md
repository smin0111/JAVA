# Java Study Notes (자바의 정석 4판 15장 정리 - 입출력 I/O)

---

# 15장 한 줄 요약

자바에서 데이터를 읽고 쓰는 모든 기술이다.

핵심 구조:
1. 스트림 기반 I/O
2. 바이트 스트림 vs 문자 스트림
3. 보조 스트림 (버퍼 등)
4. 파일 입출력
5. 직렬화

---

# 1. I/O 기본 개념

I/O = Input / Output
프로그램 <-> 외부(파일, 키보드, 네트워크) 사이의 데이터 이동을 처리하는 기술이다.
자바는 모든 입출력을 "스트림(Stream)"이라는 통로 개념으로 처리한다.

14장의 Stream과 헷갈리지 말 것:
- 14장 Stream = 컬렉션 데이터를 함수형 파이프라인으로 처리하는 기술
- 15장 Stream = 데이터가 흘러다니는 실제 입출력 통로

---

# 2. 스트림 종류 핵심 구분

가장 중요한 구분 기준:

| 구분 | 처리 대상 | 클래스 이름 |
|---|---|---|
| 바이트 스트림 | 모든 파일 (이미지, 영상, exe 등 바이너리) | InputStream / OutputStream |
| 문자 스트림 | 텍스트 파일 (한글 처리 가능) | Reader / Writer |

빠른 암기법:
텍스트 파일 -> Reader / Writer
그 외 모든 파일 -> InputStream / OutputStream

---

# 3. 바이트 스트림 (InputStream / OutputStream)

자바 입출력 계층의 최상위 부모 클래스다.

InputStream: 외부에서 프로그램으로 읽어 들이는 통로
OutputStream: 프로그램에서 외부로 내보내는 통로

파일 읽기 예제 (바이트):
```java
// "test.txt" 파일을 바이트 단위로 읽어오는 통로를 연다
FileInputStream fis = new FileInputStream("test.txt");
int data;

// read()는 1바이트씩 읽으며, 파일 끝에 도달하면 -1을 반환한다
// -1이 될 때까지 반복문을 돌린다
while((data = fis.read()) != -1){
    // 읽어온 바이트 숫자를 char 타입으로 형변환해서 출력한다
    System.out.print((char)data);
}

// 스트림을 다 쓰면 반드시 닫아서 자원을 반납해야 한다
fis.close();
```

파일 쓰기 예제 (바이트):
```java
// "test.txt" 파일과 연결된 출력 통로를 연다
FileOutputStream fos = new FileOutputStream("test.txt");
// 숫자 65는 ASCII 코드로 'A'다. 이처럼 바이트 스트림은 숫자를 넘긴다.
fos.write(65);
fos.close();
```

---

# 4. 문자 스트림 (Reader / Writer)

텍스트 전용이며 한글/유니코드 처리가 가능하다.
실무에서 로그, 설정파일, CSV 처리 등에 폭넓게 쓰인다.

Reader: 텍스트 파일 읽기
Writer: 텍스트 파일 쓰기

파일 읽기 예제 (문자):
```java
// 문자 기반으로 "test.txt" 파일을 읽는 통로를 연다
FileReader fr = new FileReader("test.txt");
int data;

// 바이트 스트림과 사용 방법은 동일하지만, 내부적으로 문자 단위로 처리해 한글이 깨지지 않는다
while((data = fr.read()) != -1){
    System.out.print((char)data);
}
fr.close();
```

---

# 5. 보조 스트림 (엄청 중요)

기존의 기본 입출력 스트림 위에 덧씌워서 속도 향상이나 편의 기능을 추가해주는 래퍼 스트림이다.
단독으로 존재할 수 없고, 반드시 기존 스트림을 감싸서 사용해야 한다.

대표적인 보조 스트림:
- BufferedInputStream / BufferedOutputStream: 버퍼를 이용해 속도를 개선한다
- BufferedReader / BufferedWriter: 텍스트 처리 시 가장 많이 쓰인다
- DataInputStream: 자바 기본 자료형(int, long 등)을 그대로 읽는다
- ObjectInputStream / ObjectOutputStream: 직렬화된 객체를 읽고 쓴다

BufferedReader (시험 단골 및 실무 핵심):
```java
// new FileReader("test.txt"): 파일과 연결된 기본 스트림을 먼저 생성한다
// new BufferedReader(...): 위 스트림을 보조 스트림으로 한 번 더 감싸서 버퍼링 기능을 추가한다
BufferedReader br = new BufferedReader(new FileReader("test.txt"));

String line;
// readLine()은 파일을 1바이트씩이 아니라 "한 줄" 단위로 통째로 읽어오는 기능이다
// 더 이상 읽을 줄이 없으면(파일 끝) null을 반환한다
while((line = br.readLine()) != null){
    System.out.println(line);
}
```

---

# 6. try-with-resources (실무 필수)

스트림은 다 쓴 뒤 반드시 close()를 호출해야 운영체제 자원 누수가 없다.
자바 7부터 try 괄호 안에 자원을 선언하면 중괄호 블록이 끝날 때 자동으로 close()를 호출해준다.

옛날 방식 (close 누락 위험):
```java
FileReader fr = null;
try {
    fr = new FileReader("a.txt");
    // 파일 처리 로직
} finally {
    // finally 블록에서 직접 닫아줘야 해서 코드가 지저분해진다
    if(fr != null) fr.close();
}
```

현대 방식 (try-with-resources):
```java
// try 괄호 안에 자원을 선언하면, 블록이 끝날 때 자동으로 close()가 호출된다
try(FileReader fr = new FileReader("a.txt")){
    // 파일 처리 로직
} catch(Exception e) {
    e.printStackTrace();
}
// 밖으로 나오는 순간 fr.close() 자동 실행
```

---

# 7. File 클래스 (파일 정보 관리)

파일 또는 디렉터리의 정보를 확인하고, 생성/삭제 등의 관리 작업을 수행하는 클래스다.

```java
// 특정 경로의 파일을 나타내는 File 객체를 생성한다 (이 시점에 실제 파일이 만들어지지는 않는다)
File f = new File("test.txt");

f.exists();       // 해당 경로에 파일이나 디렉터리가 실제로 존재하는지 확인한다
f.length();       // 파일 크기를 바이트 단위로 반환한다
f.delete();       // 파일을 물리적으로 삭제한다
f.isFile();       // 경로가 파일인지 확인한다
f.isDirectory();  // 경로가 디렉터리(폴더)인지 확인한다
```

---

# 8. 직렬화 (Serialization) - 매우 중요

직렬화: 자바 객체를 바이트 스트림으로 변환해서 파일에 저장하거나 네트워크로 전송하는 기술이다.
역직렬화: 바이트 스트림을 다시 자바 객체로 복원하는 기술이다.

활용처: Spring 세션 관리, Redis 캐시, 네트워크 통신 객체 전달 등

직렬화 조건:
직렬화하고 싶은 클래스는 반드시 Serializable 인터페이스를 구현해야 한다.
```java
// Serializable은 몸통이 비어있는 마커 인터페이스다
// 이것을 구현함으로써 "이 객체는 직렬화 가능하다"고 JVM에 표시하는 것이다
class User implements Serializable {
    String name;
    int age;
}
```

객체 저장 (직렬화):
```java
// FileOutputStream: user.dat 파일과 연결된 바이트 출력 통로
// ObjectOutputStream: 위 통로를 감싸서 자바 객체를 바이트로 변환해주는 보조 스트림
ObjectOutputStream oos =
    new ObjectOutputStream(new FileOutputStream("user.dat"));

// writeObject()가 user 객체를 바이트로 쪼개서 파일에 써준다
oos.writeObject(user);
oos.close();
```

객체 읽기 (역직렬화):
```java
// ObjectInputStream: 파일에서 읽어온 바이트를 다시 자바 객체로 조립해주는 보조 스트림
ObjectInputStream ois =
    new ObjectInputStream(new FileInputStream("user.dat"));

// readObject()는 Object 타입으로 반환하므로 원래 타입(User)으로 명시적 형변환이 필요하다
User u = (User) ois.readObject();
ois.close();
```

transient 키워드 (시험 단골):
```java
class User implements Serializable {
    String name;
    int age;
    
    // transient를 붙인 필드는 직렬화에서 완전히 제외된다
    // 파일에 저장하거나 네트워크로 전송할 때 비밀번호가 노출되는 것을 방지할 때 쓴다
    transient String password;
}
```

---

# 9. 실무 코드 예시 모음

## 1. 서버 로그 파일 작성

Spring 서버는 회원가입, 에러 등 중요 이벤트를 로그 파일로 기록한다.

```java
import java.io.*;

public class LogService {

    public static void writeLog(String msg) {
        // try-with-resources로 자원을 안전하게 관리한다
        // new FileWriter("server.log", true): true는 파일에 이어쓰기(append) 옵션이다. false면 덮어쓴다
        // BufferedWriter: FileWriter를 감싸서 버퍼 기능을 추가해 성능을 높인다
        try(BufferedWriter bw = new BufferedWriter(new FileWriter("server.log", true))) {
            bw.write(msg);    // 전달받은 메시지를 파일에 기록한다
            bw.newLine();     // 운영체제 종류에 상관없이 올바른 줄바꿈 기호를 입력해준다
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}

// 사용:
// LogService.writeLog("회원가입 성공: user123");
```

---

## 2. CSV 파일 읽기 및 객체 변환 (엑셀 업로드 기능)

관리자 페이지에서 Excel로 회원을 대량 등록할 때 사용하는 패턴이다.

users.csv:
```
kim,20
lee,25
park,30
```

```java
import java.io.*;
import java.util.*;

class User {
    String name;
    int age;

    User(String name, int age){
        this.name = name;
        this.age = age;
    }
}

public class CsvService {

    public static List<User> loadUsers() throws Exception {
        List<User> list = new ArrayList<>();

        // try-with-resources로 BufferedReader를 사용해 "users.csv" 파일을 열었다
        try(BufferedReader br = new BufferedReader(new FileReader("users.csv"))) {
            String line;
            while((line = br.readLine()) != null){ // 파일을 한 줄씩 읽는다
                // 콤마(,)를 기준으로 줄을 쪼개서 배열로 만든다
                String[] arr = line.split(",");
                // 이름(arr[0])과 나이(arr[1])를 파싱해 User 객체를 만들어 리스트에 추가한다
                list.add(new User(arr[0], Integer.parseInt(arr[1])));
            }
        }
        return list;
    }
}
```

활용처: 엑셀 업로드, 데이터 마이그레이션, 대량 회원 등록

---

## 3. 파일 업로드 저장 로직 (Spring MultipartFile 원리)

사용자가 프로필 사진을 업로드할 때 서버에 바이트 단위로 저장하는 핵심 패턴이다.

```java
public void saveFile(InputStream is) throws Exception {

    // "profile.jpg" 파일로 연결되는 출력 스트림을 try-with-resources로 열었다
    try(FileOutputStream fos = new FileOutputStream("profile.jpg")) {

        // 1024바이트(1KB) 단위로 데이터를 읽어서 일괄 처리하는 버퍼 배열이다
        // 1바이트씩 처리하면 너무 느리므로 덩어리째 읽어서 성능을 높인다
        byte[] buffer = new byte[1024];
        int len;

        // is.read(buffer): buffer에 최대 1024바이트를 읽어 채우고, 실제 읽은 바이트 수를 반환한다
        // 파일 끝에 도달하면 -1을 반환하므로 그때 반복을 멈춘다
        while((len = is.read(buffer)) != -1){
            // 버퍼에 담긴 내용을 0번 위치부터 실제 읽은 len 바이트만큼만 파일에 기록한다
            fos.write(buffer, 0, len);
        }
    }
}
// Spring의 MultipartFile.transferTo()와 MultipartFile.getInputStream() 내부가 바로 이 원리로 동작한다
```

---

## 4. 설정 파일 읽기 (application.properties 원리)

Spring Boot가 서버 실행 시 application.properties를 읽어오는 원리가 바로 이것이다.

config.txt:
```
db.url=localhost
db.user=root
db.password=1234
```

```java
import java.io.*;
import java.util.*;

public class ConfigLoader {

    public static Map<String, String> load() throws Exception {
        Map<String, String> map = new HashMap<>();

        try(BufferedReader br = new BufferedReader(new FileReader("config.txt"))) {
            String line;
            while((line = br.readLine()) != null){
                // "="를 기준으로 키와 값을 분리한다 (db.url / localhost)
                String[] arr = line.split("=");
                // arr[0]이 키, arr[1]이 값이다
                map.put(arr[0], arr[1]);
            }
        }
        return map;
    }
}
// 이 원리가 확장된 것이 Spring의 application.properties, .yml 파싱 기술이다
```

---

## 5. 객체 파일 캐싱 (직렬화 실무 활용)

DB 조회 결과가 비싼 경우, 결과 객체를 파일에 직렬화해두고 재사용하는 캐싱 패턴이다.
Redis의 원리와 동일하며, Redis는 네트워크 너머로 이 작업을 더 정교하게 수행할 뿐이다.

```java
class Product implements Serializable {
    String name;
    int price;
}

// 객체를 파일에 직렬화하여 저장
public static void save(Product p) throws Exception {
    try(ObjectOutputStream oos =
        new ObjectOutputStream(new FileOutputStream("cache.dat"))) {
        oos.writeObject(p); // 프로덕트 객체를 바이트로 변환해 cache.dat에 저장한다
    }
}

// 파일에서 역직렬화하여 객체로 복구
public static Product load() throws Exception {
    try(ObjectInputStream ois =
        new ObjectInputStream(new FileInputStream("cache.dat"))) {
        // Object 타입으로 반환되므로 Product 타입으로 강제 형변환한다
        return (Product) ois.readObject();
    }
}
```

활용처: Redis 원리 이해, 세션 저장, 서버 캐시 구현

---

# 15장 최종 핵심 요약

스트림 종류:
- 바이트 파일 -> InputStream / OutputStream
- 텍스트 파일 -> Reader / Writer

보조 스트림:
- BufferedReader / BufferedWriter를 이용해 성능과 편의성을 높인다

필수 문법 및 개념:
- try-with-resources: 스트림 자원을 자동으로 닫아준다
- File 클래스: 파일 정보 조회 및 생성/삭제 관리
- 직렬화 (Serializable): 객체를 파일 또는 네트워크로 전송 가능하게 만드는 기술
- transient: 직렬화 시 포함하지 않을 민감한 필드에 붙이는 키워드

실무 탄탄 연결 정리:
| 실무 상황 | 주로 사용하는 기술 |
|---|---|
| 서버 로그 파일 기록 | BufferedWriter + FileWriter |
| 엑셀/CSV 업로드 파싱 | BufferedReader + split() |
| 이미지, 영상 업로드 저장 | InputStream + FileOutputStream |
| 설정파일 읽기 | BufferedReader + Map |
| 객체 캐시/세션 저장 | ObjectOutputStream (직렬화) |

Spring 내부의 요청 처리, 파일 저장, 세션 관리, 캐싱 등은 전부 이 15장의 I/O 기술 위에서 동작한다.

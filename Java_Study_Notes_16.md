# Java Study Notes (자바의 정석 4판 16장 정리 - 네트워킹)

---

# 16장 한 줄 요약

자바로 서버를 만드는 기초 기술이다.
이 장을 배우면 채팅 프로그램, 파일 전송, 간단한 웹서버, 클라이언트/서버 구조 원리를 이해할 수 있다.
Spring이 내부에서 어떻게 HTTP 요청을 받고 응답하는지의 근본 원리가 바로 이 장이다.

---

# 1. 네트워크 기본 개념

컴퓨터끼리 통신하려면 3가지가 필요하다.

1. IP 주소: 네트워크 상에서 컴퓨터를 식별하는 고유 주소 (예: 192.168.0.1)
2. Port 번호: 한 컴퓨터 안에서 실행 중인 특정 프로그램(프로세스)을 식별하는 번호
3. 프로토콜: 통신 규칙 (TCP / UDP)

자주 쓰이는 포트 번호:
| 포트 | 의미 |
|---|---|
| 80 | HTTP 웹 서버 |
| 443 | HTTPS 웹 서버 |
| 3306 | MySQL 데이터베이스 |
| 8080 | Spring Boot 기본 포트 |

서버 접속 = IP 주소 + Port 번호의 조합으로 특정 컴퓨터의 특정 프로그램에 연결한다.

---

# 2. 네트워크 통신 방식 2가지 (시험/면접 단골)

| 구분 | TCP | UDP |
|---|---|---|
| 연결 방식 | 연결형 (사전에 연결 확립) | 비연결형 (그냥 던짐) |
| 속도 | 느림 | 빠름 |
| 신뢰성 | 높음 (데이터 보장) | 낮음 (손실 가능) |
| 데이터 순서 보장 | O | X |

TCP 특징:
데이터의 순서와 완전성이 보장된다. 빠진 조각이 있으면 다시 요청해서 받아오는 재전송 과정이 내장되어 있어 신뢰도가 높다. 대신 이 과정 때문에 속도가 다소 느리다.
주 사용처: 웹 브라우저, 로그인, 파일 전송, 채팅. 우리가 만드는 Spring 서버도 모두 TCP 기반이다.

UDP 특징:
속도가 매우 빠르지만 데이터 유실 가능성이 있다. 약간 끊겨도 실시간성이 더 중요한 경우에 쓴다.
주 사용처: 게임, 동영상 스트리밍, 실시간 방송

---

# 3. 자바 네트워크 핵심 클래스 2개

이 2가지가 자바 네트워크 통신의 전부다.

```java
ServerSocket  // 서버 측: 클라이언트의 접속을 기다리는 창구
Socket        // 클라이언트 측: 실제 통신이 이루어지는 연결 통로
```

---

# 4. 서버 기본 구조와 흐름 (반드시 암기 수준)

서버 구동의 3단계:
1. ServerSocket 생성 (특정 포트에서 문 열기)
2. 클라이언트 접속 대기 (accept 호출 - 이 시점에서 프로그램이 멈추고 기다림)
3. 연결된 Socket으로 데이터 송수신

서버 코드 예제:
```java
import java.net.*;
import java.io.*;

public class Server {
    public static void main(String[] args) throws Exception {
        // 7777번 포트를 열고 클라이언트가 접속하기를 기다리는 문을 만든다
        ServerSocket server = new ServerSocket(7777);
        System.out.println("서버 시작 - 접속 대기 중");

        // accept()는 클라이언트가 연결 요청을 보낼 때까지 이 줄에서 프로그램이 멈춘다 (블로킹)
        // 누군가 접속하면 그 클라이언트와 연결된 Socket 객체를 반환한다
        Socket socket = server.accept();
        System.out.println("클라이언트 접속 완료!");

        // 연결된 소켓에서 데이터를 읽을 수 있는 통로(InputStream)를 가져온다
        InputStream in = socket.getInputStream();
        byte[] arr = new byte[100];
        // 클라이언트가 보낸 바이트 데이터를 arr 배열에 읽어 담는다
        in.read(arr);

        // 읽어온 바이트 배열을 문자열로 변환해 출력한다
        System.out.println(new String(arr));
    }
}
```

클라이언트 코드 예제:
```java
import java.net.*;
import java.io.*;

public class Client {
    public static void main(String[] args) throws Exception {
        // "localhost"의 7777번 포트에 접속 요청을 보낸다
        // 서버의 ServerSocket이 accept()로 대기 중이어야 연결이 된다
        Socket socket = new Socket("localhost", 7777);

        // 연결된 소켓에서 데이터를 보낼 수 있는 통로(OutputStream)를 가져온다
        OutputStream out = socket.getOutputStream();
        // 문자열을 바이트 배열로 변환해서 서버로 전송한다
        out.write("안녕 서버".getBytes());
    }
}
```

통신 흐름:
```
Client                    Server
Socket         ->         ServerSocket (포트 대기)
(연결 요청)    ->         accept() (연결 수락 후 Socket 반환)
write()        ->         read()   (데이터 전송/수신)
```

---

# 5. 스트림 기반 문자 통신으로 업그레이드

위에서는 byte 배열로 데이터를 주고받았는데, 실무에서는 이 방식을 거의 안 쓴다.
실제 채팅이나 텍스트 프로토콜 통신에서는 줄 단위로 데이터를 주고받는 문자 스트림 방식을 사용한다.

소켓 스트림 변환 구조 (시험/면접 단골):
소켓이 제공하는 InputStream/OutputStream을 한 단계씩 감싸서 문자 스트림으로 변환한다.

```
수신: InputStream -> InputStreamReader -> BufferedReader (readLine() 사용 가능)
송신: OutputStream -> PrintWriter (println() 사용 가능)
```

서버 업그레이드 버전 (문자 통신):
```java
import java.net.*;
import java.io.*;

public class Server {
    public static void main(String[] args) throws Exception {
        ServerSocket server = new ServerSocket(7777);
        Socket socket = server.accept();

        // 소켓의 InputStream을 문자 스트림(BufferedReader)으로 변환한다
        // socket.getInputStream(): 소켓에서 바이트 입력 통로를 가져온다
        // new InputStreamReader(...): 바이트 스트림을 문자 스트림으로 변환한다
        // new BufferedReader(...): 문자 스트림에 버퍼를 추가해서 readLine() 기능을 쓸 수 있게 한다
        BufferedReader in =
            new BufferedReader(
                new InputStreamReader(socket.getInputStream()));

        // 소켓의 OutputStream을 PrintWriter로 감싸서 println()으로 줄 단위 전송이 가능하게 한다
        // 두 번째 인자 true는 println() 호출 시 자동으로 flush(전송)하게 하는 옵션이다
        PrintWriter out =
            new PrintWriter(socket.getOutputStream(), true);

        // 클라이언트가 보낸 한 줄의 문자열을 읽는다
        String msg = in.readLine();
        System.out.println("클라이언트: " + msg);

        // 클라이언트에게 응답 메시지를 한 줄 보낸다
        out.println("반가워!");
    }
}
```

클라이언트 업그레이드 버전:
```java
import java.net.*;
import java.io.*;

public class Client {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("localhost", 7777);

        // 서버와 동일하게 소켓 스트림을 문자 스트림으로 변환한다
        BufferedReader in =
            new BufferedReader(
                new InputStreamReader(socket.getInputStream()));

        PrintWriter out =
            new PrintWriter(socket.getOutputStream(), true);

        // 서버에게 문자 메시지를 한 줄 전송한다
        out.println("안녕 서버");
        // 서버의 응답 한 줄을 읽어서 출력한다
        System.out.println("서버: " + in.readLine());
    }
}
```

이 구조로 양방향 문자 통신이 완성된다. 여기서 반복문을 추가하면 기초 채팅 프로그램이 된다.

---

# 6. 멀티쓰레드 서버 (핵심 중의 핵심)

현재 서버의 가장 큰 문제점:
```java
socket = server.accept(); // 한 명 접속하면
// 이 한 명을 처리하는 동안 다른 사람은 전혀 접속을 못 한다
```

한 명만 처리할 수 있는 서버는 실제 서비스가 불가능하다.
해결책: 클라이언트 1명이 접속할 때마다 전담 쓰레드를 1개씩 생성해서 맡긴다.

```
클라이언트 1  ->  Thread 1
클라이언트 2  ->  Thread 2     <- 모두 동시에 처리
클라이언트 3  ->  Thread 3
```

1단계: 클라이언트 전담 쓰레드 클래스 작성
```java
class ClientThread extends Thread {
    // 이 쓰레드가 담당할 클라이언트의 소켓을 보관하는 필드
    Socket socket;

    ClientThread(Socket socket){
        this.socket = socket;
    }

    @Override
    public void run(){
        try{
            // 전담 소켓의 스트림을 문자 스트림으로 변환한다
            BufferedReader in =
                new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));

            PrintWriter out =
                new PrintWriter(socket.getOutputStream(), true);

            String msg;
            // 클라이언트가 연결을 끊을 때까지(null 반환) 계속 메시지를 주고받는다
            while((msg = in.readLine()) != null){
                System.out.println("수신: " + msg);
                // 받은 메시지 앞에 "Echo: "를 붙여서 그대로 돌려보낸다
                out.println("Echo: " + msg);
            }
        }catch(Exception e){
            // 클라이언트가 연결을 끊으면 예외가 발생하므로 별도 처리 없이 종료
        }
    }
}
```

2단계: 서버 메인 로직에서 접속마다 쓰레드 생성
```java
public class Server {
    public static void main(String[] args) throws Exception {
        ServerSocket server = new ServerSocket(7777);
        System.out.println("멀티쓰레드 서버 시작");

        // 서버는 무한루프를 돌면서 계속해서 클라이언트의 접속을 기다린다
        while(true){
            // 누군가 접속하면 즉시 전담 쓰레드를 하나 생성한 뒤 start()로 독립 실행시킨다
            // 메인 쓰레드는 다시 위로 올라가 다음 클라이언트를 accept()로 기다린다
            Socket socket = server.accept();
            new ClientThread(socket).start();
        }
    }
}
```

이 구조가 실제 네트워크 서버의 핵심 아키텍처다.
Spring Boot의 Tomcat도 내부적으로 이와 유사한 방식으로 요청마다 쓰레드를 할당해서 동시에 처리한다.

(Blue 백엔드 연관 개념):
Spring의 `@RestController`에 요청이 들어오면, Tomcat이 이 멀티쓰레드 서버 구조와 똑같은 방식으로 요청당 쓰레드를 하나씩 배정해서 컨트롤러 메서드를 실행시킨다. 13장의 멀티쓰레드와 이 장의 네트워크 소켓이 합쳐진 결과가 바로 Spring 서버다.

---

# 7. 16장 최종 핵심 요약

TCP 서버의 3대 구성 요소 (반드시 암기):
```
ServerSocket = 클라이언트의 접속 요청을 받아내는 창구
Socket       = 클라이언트와 실제 데이터를 주고받는 통신 통로
Thread       = 여러 클라이언트를 동시에 처리하기 위한 병렬 실행 단위
```

TCP vs UDP 핵심 차이:
- TCP: 연결형, 데이터 순서/완전성 보장, 속도 느림. 웹/로그인/채팅 등에 사용
- UDP: 비연결형, 데이터 유실 가능, 속도 빠름. 게임/스트리밍 등에 사용

소켓 스트림 변환 패턴 (시험 단골):
```
InputStream  -> InputStreamReader -> BufferedReader  (readLine으로 줄 단위 수신)
OutputStream -> PrintWriter                          (println으로 줄 단위 송신)
```

Spring과의 연결:
- Spring Boot가 8080포트를 열고 요청을 기다리는 것 = ServerSocket
- HTTP 요청이 들어오는 것 = Socket 연결
- 요청마다 컨트롤러가 돌아가는 것 = 멀티쓰레드 처리
- 이 3가지가 곧 16장의 내용이다.

---

# 8. 실무 연결 예시 - RestTemplate (Blue 백엔드의 실제 네트워크 통신)

직접 Socket을 열어서 통신하는 것은 자바 순수 네트워크 레벨의 기술이다.
실무의 Spring Boot 환경에서는 이 과정을 모두 추상화한 `RestTemplate`이라는 클래스를 사용해서 외부 API와 HTTP 통신을 처리한다.
개념 구도: 16장의 (Socket + InputStream + OutputStream) = Spring의 RestTemplate

Blue-back 실제 사용 코드 (KakaoLocalService.java):
카카오 지도 REST API에 네트워크 요청을 보내서 위치 좌표를 받아오는 로직이다.

```java
// RestTemplate: 자바의 Socket, InputStream, OutputStream 기반 HTTP 통신을 
// Spring이 한 줄로 쓸 수 있게 랩핑해둔 유틸리티 클래스다
private final RestTemplate restTemplate = new RestTemplate();

// 카카오 지도 API에 GET 요청을 보내고 JSON 응답을 Map 으로 받아오는 공통 헬퍼 메서드
private ResponseEntity<Map> callKakaoAPI(URI uri) {
    try {
        // HTTP 요청 헤더에 카카오 인증키를 담는다 (카카오 API 규약)
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "KakaoAK " + kakaoApiKey);

        // 헤더만 담은 빈 요청 엔티티를 만든다 (GET 요청이라 body는 비워둔다)
        HttpEntity<Void> entity = new HttpEntity<>(headers);

        // restTemplate.exchange() 내부에서 실제로 일어나는 일:
        // 1. 카카오 서버 IP를 DNS로 조회
        // 2. 443 포트(HTTPS)로 Socket 연결 수립
        // 3. OutputStream으로 HTTP GET 요청 패킷 전송
        // 4. InputStream으로 JSON 응답 패킷 수신
        // 5. 수신된 바이트를 문자열로 변환 후 Map 객체로 파싱해서 반환
        // 이 복잡한 과정 전부를 이 한 줄이 대신 처리해준다.
        ResponseEntity<Map> response =
            restTemplate.exchange(uri, HttpMethod.GET, entity, Map.class);

        return response;

    } catch (Exception e) {
        System.out.println("카카오 API 호출 중 에러 발생: " + e.getMessage());
        return null;
    }
}

// 사용자 입력 주소를 위도/경도 좌표로 변환하는 메서드 (카카오 주소 검색 API 호출)
private double[] searchByAddressAPI(String input) {
    // 한글 주소의 URL 인코딩을 자동 처리해주는 빌더로 URI를 만든다
    URI uri = UriComponentsBuilder
        .fromUriString("https://dapi.kakao.com/v2/local/search/address.json")
        .queryParam("query", input)
        .encode(StandardCharsets.UTF_8)
        .build()
        .toUri();

    ResponseEntity<Map> response = callKakaoAPI(uri);

    // 응답 JSON에서 "documents" 배열의 첫 번째 결과물의 위도(y)와 경도(x)를 추출한다
    List<Map<String, Object>> docs =
        (List<Map<String, Object>>) response.getBody().get("documents");

    if (docs != null && !docs.isEmpty()) {
        Map<String, Object> doc = docs.get(0);
        return new double[] {
            Double.parseDouble(doc.get("y").toString()), // 위도 (latitude)
            Double.parseDouble(doc.get("x").toString())  // 경도 (longitude)
        };
    }
    return null;
}
```

16장 개념과의 연결 정리:
- `new RestTemplate()` = 서버와 통신할 준비를 하는 것  (Socket 객체 생성과 동일)
- `restTemplate.exchange()` = 서버로 요청을 보내고 응답을 받는 것 (write & read와 동일)
- `HttpHeaders` = 16장에서 OutputStream에 헤더 정보를 직접 써서 보내야 했던 부분을 추상화한 것
- `ResponseEntity<Map>` = 16장에서 InputStream으로 읽어온 바이트를 파싱한 결과와 동일

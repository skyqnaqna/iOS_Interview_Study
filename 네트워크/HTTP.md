# 답변

> HTTP는 클라이언트-서버 구조로 동작하며, 무상태 프로토콜을 지향하며 비연결성인 특징을 갖고 있습니다.
HTTP 메세지로 통신하며 단순하고 확장 가능합니다.
> 

# 내용

## 1. 클라이언트-서버구조

클라이언트가 서버에 요청(Request)을 보내면, 서버는 응답(Response)를 대기하는 구조로 되어있다.

서버가 요청에 대한 결과를 만들어서 응답한다.

## 2. 무상태(Stateless) 프로토콜

### Stateless

- 서버와 클라이언트 간의 각 통신은 서로 독립적이다.
- 서버는 클라이언트의 상태를 보존하지 않아 단순히 요청이 오면 응답을 보내는 역할
- 세션관리는 클라이언트의의 책임
- 대표 프로토콜 :  UDP, HTTP
- 예시를 보면, A가 Client고 B는 Server로 A가 B에게 자신의 요청정보를 계속 누적해서 전달한다. B는 단순히 A의 요청에 응답만 할 뿐 A의 요청정보를 따로 저장하진 않는다. 
A가 자신의 요청사항을 저장하고 있기 때문에 어떤 점원이 오거나 바뀌어도 상관없이 원하는 응답을 받을 수 있다. 즉, 갑자기 클라이언트 요청이 증가해도 점원(서버)를 여러 투입할 수 있다.
- 즉, Stateless는 응답 서버를 쉽게 바꿀 수 있어 무한한 서버 증설이 가능하다!

### Stateful

- 클라이언트와의 세션의 상태에 기반하여 서버는 response를 보낸다.
- 세션정보를 서버에 저장한다.
- 중단되면 컨텍스트와 기로 서버에 저장되기 때문에 중단되어도 중단된 곳 부터 다시 시작할 수 있다.
- 대표 프로토콜 : TCP (3-way-handshaking)
- 같은 점원일때는 B가 A의 요청사항을 갖고 있어 문제가 되지 않지만, 만약 B 점원이 아파서(B 서버가 고장나서) C, D 점원(서버)으로 교체되어야 한다면 A의 이전 요청사항을 알 수 없어 응답을 제대로 보낼 수 없다.
중간에 점원이 바뀌면 안되고, 바뀔 때 교체될 점원에게 상태 정보를 미리 알려야한다.



|    | 예시 |
| ------- | ------ |
| Stateless | <img width="730" alt="스크린샷 2022-11-14 오후 2 00 03" src="https://user-images.githubusercontent.com/37897873/201578943-ca8e5422-92db-4ac9-ac68-851cbdb21ae8.png"> |
| Stateful |  <img width="730" alt="스크린샷 2022-11-14 오후 2 00 53" src="https://user-images.githubusercontent.com/37897873/201578988-c80786fa-cb32-4d5b-9a60-c2c79a852442.png"> |



## 3. 비연결성

- HTTP는 기본적으로 연결을 유지하지 않는다.
- 서버 자원을 매우 효율적으로 사용할 수 있다.
- 현재 요청을 받을때만 자원을 연결하고 끝나면 끊어서 최소한의 자원만 사용한다.
- if 연결성이라면, 연결된 클라이언트와 모두 연결하고 있어야 하므로 사용되지 않을때도 연결하고 있어 자원이 소모된다.

## 4. HTTP 메세지
<img width="330" alt="스크린샷 2022-11-09 오후 4 48 41" src="https://user-images.githubusercontent.com/37897873/201579625-3a5aacd9-8bd0-4dae-82eb-38a41bc87b36.png">

- start-line(=request line) : HTTP 메서드 ex) `GET` , 요청대상(`/search?q=hello`), HTTP Version, 응답메세지 ex) `200`
- header : HTTP 전송에 필요한 모든 부가정보가 들어가 있음
    - 형식 : fieldname : OWS(띄어쓰기 허용) field-value OWS ex) `HOST: www.naver.com`
- empty line : 공백라인은 필수이다.
- message body : 실제 전송할 데이터가 들어감 / HTML, 이미지, 영상 등등

# 추가 질문

## HTTP란?

HTTP(HyperText Transfer Protocol)은 웹 브라우저와 웹서버 사이의 통신을 위한 표준 프로토콜이다.

이전에는 주로 HTML 문서를 주고 받기위해 사용했지만, 지금은 거의 모든 형태의 데이터를 전송 가능하다.

- TCP : HTTP/1.1, HTTP/2
- UDP : HTTP/3

## HTTP1.0/1.1/2.0의 각 차이는 무엇인가요?

### 1. HTTP1.0

기본적으로 한 연결당 하나의 요청을 처리하도록 하여 RTT(Round Trip Time) 증가를 불러오게 설계되었다. 즉, 서버로부터 파일을 가져올 때마다 TCP 3-way-handshaking을 해야 하기 때문에 패킷왕복시간인 RTT가 증가할 수 밖에 없다.
RTT가 증가하면 클라이언트가 응답받는 시간이 길어지고 서버에도 부담이 많이 간다. 이를 해결하기위해 이미지 스플리팅, 코드압축, 이미지 인코딩 방식을 통해 해결했다고 한다.

<img width="199" alt="스크린샷 2022-11-09 오후 5 04 15" src="https://user-images.githubusercontent.com/37897873/201579664-5a1859fc-a8a0-4066-8b0d-ca44e98a5a3c.png">


### 2. HTTP 1.1 (가장 많이 사용)

한 번 TCP 초기화를 한 이후에 keep-alive 옵션으로 여러개의 파일을 송수신할 수 있게 바뀌었다. 1.0과의 차이는 TCP 세션을 지속적으로 유지할 수 있는가? 이다.
Server는 Client의 요청을 수용하고 응답 이전까지 TCP 연결을 끊지 않고 유지한다.
이미 연결되어 있는 TCP 연결을 유지하여 재사용하기에 여러번의 Handshake 과정이 생략되어 자원을 절약할 수 있고 네트워크 혼잡을 줄일 수 있다.

- 파이프라이닝(Pipelining)
  HTTP 1.1에서는 Client와 Server의 요청/응답의 효율성을 개선하기 위해 파이프라이닝이 등장했는데, 쉽게말해 여러 요청들에 대한 응답 처리를 한번에 처리하기 위해 뒤로 미루는 방법이다.
  즉, 클라이언트는 요청에 대한 응답을 기다리지 않고, 여러개의 HTTP Reqeust를 하나의 TCP/IP Packet으로 연속적으로 Packing 해서 요청한다.
    
### 2-1. HOL Blocking (Head Of Line Blocking)
한번에 한 파일밖에 보내지 못했기 때문에, 같은 큐에 있는 특정 파일의 로딩이 지연되면 다른파일 로딩들도 줄줄이 느려지는 성능저하 즉, 병목현상이 발생한다.
<img width="538" alt="스크린샷 2022-11-09 오후 5 51 50" src="https://user-images.githubusercontent.com/37897873/201579737-a927f2f5-c2ae-41fd-87b3-e92044bfd344.png">


### 2-2. 무거운 Header

헤더에 많은 메타 정보들과 실제 전송정보 등 많은 정보가 저장되어 있는데 이를 매 요청시에 전송해야 한다. 예를 들어, hello 한 단어를 보내기 위해 아래와 같은 메세지보다 헤더가 더 큰 경우가 발생된다.

<img width="616" alt="스크린샷 2022-11-09 오후 6 04 05" src="https://user-images.githubusercontent.com/37897873/201579778-ad9b0cc5-8346-4c12-9124-45a14ad8a942.png">

### 3. HTTP 2.0

HTTP1.x 보다 지연 시간을 줄이고 응답시간을 더 빠르게 하는 성능 향상에 초점이 맞춰져 있다. 

### 3-1. 멀티플렉싱 
여러 개의 스트림을 사용하여 여러개의 메세지를 동시에 송수신한다. 하나의 패킷이 손실되어도 해당 스트림에만 영향을 끼치고 나머지는 정상적으로 동작 가능하다는 장점이 있다. 
HTTP 1.1에서 문제였던 HOL Blocking을 해결하기 위해 여러 파일을 한번에 병렬 전송하여 로딩 시간을 줄이고 응답 순서에 상관없이 처리가 가능해졌다.

<img width="882" alt="스크린샷 2022-11-09 오후 6 05 25" src="https://user-images.githubusercontent.com/37897873/201579804-3413a7c1-ae6f-4eef-a1bc-38228d49023a.png">

### 3-2. 헤더 압축
헤더 정보를 압축하기 위해 Header Table과 Huffman Endcoding 기법을 사용하여 처리한다.
구체적으로 중복된 Header에 대한 정보는 index 값만 전송하고, 중복되지 않은 Header 정보의 값은 인코딩하여 전송한다.
    
### 3-3. 서버 푸쉬
HTTP/1.1 에서는 클라이언트 → 서버에 요청해야만 파일을 다운받았다면 HTTP/2에서는 클라이언트 요청 없이 서버가 바로 리소스를 푸쉬할 수 있다.
아래와 같이 이전에는 하나씩 리소스를 요청하여 받았지만, 서버푸쉬를 통해 미리 html을 읽으면서 안에 있던 css 파일을 서버에 푸시하여 클라이언트에게 먼저 전달 할 수도 있다.

<img width="481" alt="스크린샷 2022-11-09 오후 6 08 59" src="https://user-images.githubusercontent.com/37897873/201579816-917a7ec9-87b5-4469-9fea-72e2213cac43.png">

### 3-4. HTTPS
HTTP/2는 HTTPS위에서 동작한다. 애플리케이션 계층과 전송 계층 사이에 신뢰계층인 SSL/TLS 계층을 넣어 신뢰할 수 있는 HTTP 요청을 할 수 있도록 한다. ‘통신을 암호화’ 하는 것이다.

## www.naver.com을 입력하면 어떤일이 발생하나요?

<img width="500" alt="스크린샷 2022-11-09 오후 6 08 59" src="https://user-images.githubusercontent.com/37897873/201579909-ec69f76e-df55-4a99-801a-c8b13cc6528c.png">

1. 먼저 네이버 url을 입력하면, <br>
2. 네이버의 주소 중 도메인 네임 부분을 DNS 서버에 검색한다. (순차적이지 않고 뒤에서부터 서버에 검색)
    <img width="481" alt="스크린샷 2022-11-09 오후 6 08 59" src="https://user-images.githubusercontent.com/37897873/201579858-09267e42-3a9b-4b17-a949-fd0656e10255.png">
    
3. DNS 서버에서 도메인 네임의 IP 주소를 찾아 입력한 URL 정보와 전달 
4. URL 정보와 IP 주소는 HTTP Protocol을 사용해 HTTP 요청 메세지를 생성
5. TCP 프로토콜을 사용해 인터넷을 거쳐 해당 IP 주소의 네이버 서버에 요청
6. 도착한 HTTP 요청 메세지는 HTTP 프로토콜로 URL 정보로 바뀜
7. 웹 서버는 도착한 URL 정보의 데이터를 검색 후 요청에 대한 네이버 서버도 응답 페이지를 원래 컴퓨터로 보냄 
8. 도착한 HTTP 응답 메세지는 HTTP 프로토콜로 웹 페이지 데이터로 변환됨
9. 받은 브라우저는 html 데이터를 파싱하고 처리하여 naver가 사용자 브라우저에 출력된다.

### 쿠키와 세션의 차이

준비중..

# 참고 자료

- [https://www.inflearn.com/course/http-웹-네트워크/dashboard](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/dashboard)
- [https://m.blog.naver.com/PostView.naver?blogId=jhc9639&logNo=222700552159&referrerCode=0&searchKeyword=www](https://m.blog.naver.com/PostView.naver?blogId=jhc9639&logNo=222700552159&referrerCode=0&searchKeyword=www)
- [https://withbundo.blogspot.com/2021/08/httphttp2-http2.html](https://withbundo.blogspot.com/2021/08/httphttp2-http2.html)
- [https://post.naver.com/viewer/postView.nhn?volumeNo=16561296&memberNo=1834](https://post.naver.com/viewer/postView.nhn?volumeNo=16561296&memberNo=1834)
- [https://hanamon.kr/네트워크-http-http란-특징-무상태-비연결성/](https://hanamon.kr/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-http-http%EB%9E%80-%ED%8A%B9%EC%A7%95-%EB%AC%B4%EC%83%81%ED%83%9C-%EB%B9%84%EC%97%B0%EA%B2%B0%EC%84%B1/)
- [https://www.tutorialspoint.com/difference-between-stateless-and-stateful-protocols](https://www.tutorialspoint.com/difference-between-stateless-and-stateful-protocols)
- [https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-서비스와-HTTP-및-REST](https://5equal0.tistory.com/entry/StatefulStateless-Stateful-vs-Stateless-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%99%80-HTTP-%EB%B0%8F-REST)
- HTTP 1.0/1.1/2.0 : [https://haengsin.tistory.com/74](https://haengsin.tistory.com/74)
- 웹의 동작 원리 : [http://tcpschool.com/webbasic/works](http://tcpschool.com/webbasic/works)
    - [https://amunre21.github.io/web/1-site-works/](https://amunre21.github.io/web/1-site-works/)
- 우형 발표자료 : [https://www.youtube.com/watch?v=xcrjamphIp4](https://www.youtube.com/watch?v=xcrjamphIp4)
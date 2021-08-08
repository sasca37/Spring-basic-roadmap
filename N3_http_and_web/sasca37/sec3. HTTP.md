## HTTP 

- Hyper Text Transper Protocol
- HYML 같은 리소스 들을 가져올 수 있도록 해주는 프로토콜 

- HTML, TEXT, 이미지, 음성, 영상, 파일, JSON, XML (API) 등 모든 형태 데이터 전송 가능 
- 서버간에 데이터를 주고 받을 때도 대부분 HTTP를 사용한다.



## HTTP 역사

- HTTP/0.9 1991년: GET 메서드만 지원, HTTP 헤더X 
-  HTTP/1.0 1996년: 메서드, 헤더 추가 
-  **HTTP/1.1 1997년: 가장 많이 사용**
  - RFC2068 (1997) -> RFC2616 (1999) -> RFC7230~7235 (2014) 
  - RFC : Request For Comments : 비평을 기다리는 문서라는 의미로 메모형태
- HTTP/2 2015년: 성능 개선 
- HTTP/3 진행중: TCP 대신에 UDP 사용, 성능 개선
  - TCP : 1.1 / 2 버전 사용 



## HTTP 특징

- 클라이언트 서버 구조 
- 무상태 프로토콜(Stateless), 비연결성(connectionless) 
- HTTP 메시지 
- 단순함, 확장 가능





### 클라이어트-서버

![sec3  사진1](https://user-images.githubusercontent.com/81945553/128594643-0f61e939-e8c8-4487-8720-762a018d3887.png)

- 비즈니스로직, 데이터는 서버에서 다루고, 클라이언트는 UI, 사용성에 집중하여 분리 



### 무상태 프로토콜 (Stateless)

- 서버가 클라이언트의 상태를 보존하지 않는다.
  -  상태 유지의 경우는 항상 같은 서버가 유지되어야 한다. (도중에 서버가 바뀌거나 장애시 데이터를 모름)
  -  중간에 서버가 바뀌어도 확장이 가능하다. ex) 노트북, 몇개, 카드결제 등 모든정보를 클라이언트가 전달 
  -  즉, 응답 서버를 쉽게 바꿀수 있다. 
- 장점 : 서버 확장성이 높다 
  - 스케일 아웃 - 중계서버를 두고 수평확장
- 단점 : 클라이언트가 추가 데이터 전송
  - 로그인의 경우는 상태유지 설계가 필요 



### 비연결성 (connectionless)

- 서버는 연결 유지 X, 최소한의 자원 유지
  - 연결 유지시 놀고있는 클라이언트들의 연결도 유지해야되므로 낭비가 생긴다.
- HTTP는 기본이 연결을 유지하지 않는 모델
- 일반적으로 초 단위 이하의 빠른 속도로 응답

![sec3  사진2](https://user-images.githubusercontent.com/81945553/128594644-502559a4-5199-4776-a6fb-8d027d6ce39a.png)

- 단점 : 새로운 요청시 TCP/IP 연결을 새로 맺어야한다

  - 3WAY HANDSHAKE 시간 추가 

![sec3  사진3](https://user-images.githubusercontent.com/81945553/128594646-ac90b8f0-68e4-48d7-8375-4efad05c47c3.png)
  - HTTP/2, HTTP/3에서 최적화 
  - HTTP 지속연결 (Pesistent Connections)로 문제 해결 



## HTTP 메시지

![sec3  사진4](https://user-images.githubusercontent.com/81945553/128594649-01f2d497-7ffe-44e4-b65b-a70800c17e4b.png)

-  메시지 구조에 CRLF는 무조건 있어야 한다. 



### 시작라인 (요청 메시지)

- start-line = request-line(메서드 종류) / request-line 
- request-line = method SP(공백) request-target SP HTTP-version CRLF(엔터)



#### 요청 대상

- 절대경로[?쿼리] ex) GET /search?q=hello&hi=ko HTTP/1.1
- 절대경로란 "/"로 시작하는 경로를 말한다.



### 시작라인 (응답 메시지)

- start-line = request-line / status-line 
- status-line = HTTP-version SP status-code(상태코드) SP reason-phrase(상태코드 글 OK 등) CRLF
  - HTTP 상태 코드 : 요청 성공, 실패를 나타냄
  - 200 : 성공, 400 : 클라이언트 요청 오류, 500 : 서버 내부 오류



### 헤더

- HTTP 전송에 필요한 모든 부가정보
- 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트(브라우저) 정보, 서버 애플리케이션 정보, 캐시 관리 정보 등
- 필요시 임의의 헤더 추가 가능  
  - EX) helloworld: hihi

### 바디

- 실제 전송할 데이터
- HTML, 이미지, 영상, JSON 등 byte 로 표현할 수 있는 모든 데이터 전송 가능 






개발자는 평생 HTTP 기반위에서 개발한다.

한번은 HTTP 정리하자!

# 1. IP(인터넷 프로토콜)

![image](https://user-images.githubusercontent.com/38436013/126794791-643243cd-7d2a-457c-8765-c9f828c13fe2.png)

인터넷 상에서 클라이언트와 서버는 IP를 이용해 통신한다.  

> IP (인터넷 프로토콜)
>
> - 지정한 IP 주소에 패킷(데이터) 단위로 데이터 전달 
> - 패킷에는 출발지IP, 목적지 IP 등을 담고 있다. 

### IP 프로토콜 한계

- 비연결성 : 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷 전송 
- 비신뢰성 : 중간에 패킷이 분실되면? , 패킷이 순서대로 안오면?
- 프로그램 구분 : 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상이면?

# 2. TCP, UDP

IP의 한계를 TCP가 해결해준다.

### 인터넷 프로토콜 스택의 4계층

![image](https://user-images.githubusercontent.com/38436013/126796613-7bab5e62-442d-45b9-96e7-b1f9911417e7.png)

### 프로토콜 계층

![image](https://user-images.githubusercontent.com/38436013/126796645-fd516bc5-e487-42db-9c8c-75c7fe6f9002.png)

### A -> B에게 채팅한다고 가정

클라이언트는 메시지를  생성해서 소켓 라이브러리에 넘김 ->  메시지 정보에 TCP를 붙임 -> 메시지 + TCP + IP 붙임 -> LAN 통해 나갈때 이더넷프레임(MAC 등 물리적 정보)을 포함하여 나감-> 인터넷 통해 서버에 전송

### TCP/IP 패킷 정보

![image](https://user-images.githubusercontent.com/38436013/126796685-4f7a322e-490c-42cf-a210-ecb37b4b8577.png)

IP + TCP로 IP의 한계를 보완

### TCP 특징

전송 제어 프로토콜(Transmission Control Protocol)

- 연결지향 - TCP 3 way handshake (가상 연결) 
- 데이터 전달 보증 
- 순서 보장 
- 신뢰할 수 있는 프로토콜 
- 현재는 대부분 TCP 사용

### TCP 3 way handshake

![image](https://user-images.githubusercontent.com/38436013/126797055-24cfc9e2-1472-4d2b-8d1b-d314f2855109.png)

3번 까지 과정을 마치면 데이터를 전송한다.  

### 데이터 전달 보증 

![image](https://user-images.githubusercontent.com/38436013/126797079-5b5db12d-75e1-4150-a40a-b1f2edc82c89.png)

### 순서 보장 

![image](https://user-images.githubusercontent.com/38436013/126797112-c7bb22f8-cf5e-45cc-8817-a0a61bd412a7.png)

### UDP 특징

사용자 데이터그램 프로토콜(User Datagram Protocol)

- 하얀 도화지에 비유(기능이 거의 없음) 
- 연결지향 - TCP 3 way handshake X 
- 데이터 전달 보증 X 
- 순서 보장 X 
- 데이터 전달 및 순서가 보장되지 않지만, `단순하고 빠름 `
- 정리 
  - **IP와 거의 같다. +PORT +체크섬 정도만 추가**  (PORT는 패킷 구분)
  - 애플리케이션에서 추가 작업 필요

# 3. PORT

### 한번에 둘 이상 연결?

![image](https://user-images.githubusercontent.com/38436013/126798902-d8a9692c-8ce1-4890-bb0b-20868c0e40ef.png)

게임 패킷, 화상통화 패킷 구분 -> PORT

**PORT는 같은 IP 내에서 프로세스를 구분한다.**

- 0 ~ 65535 할당 가능  
- 0 ~ 1023: 잘 알려진 포트, 사용하지 않는 것이 좋음 
  - FTP - 20, 21 
  - TELNET - 23 
  - HTTP - 80 
  - HTTPS - 443

# 4. DNS 

IP를 기억하기 어렵고 변경될 수 있다. 이런 문제를 해결하기 위해 등장한 것이 DNS이다.
DNS는 도메인 네임 시스템(Domain Name System)으로, `도메인 명을 IP주소로 변환`한다.

![image](https://user-images.githubusercontent.com/38436013/126799342-1fe2cf88-e389-4667-abf3-1020610cc21a.png)
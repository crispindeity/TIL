# Web Socket

## Web Socket이란?

- 두 프로그램 간의 메시지를 교환하기 위한 통신 방법 중 하나이다.
- Client <--Message--> Server

## Web Socket의 특징

### 양방향 통신(Full-Duplex)

- 데이터 송수신을 동시에 처리 할 수 있는 통신 방법
- 클라이언트와 서버가 서로에게 원할 때 데이터를 주고 받을 수 있다.

### 실시간 네트워킹(Real Time-Networking)

- 웹 환경에서 연속된 데이터를 빠르게 노출
- 여러 단말기에 빠르게 데이터를 교환

## 웹 소켓 동작 방법(핸드 쉐이킹)

### 연결 요청시 발생되는 HTTP Header Message
```bash
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhllHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version:13
```
- GET /chat HTTP/1.1: 연결 수립 과정은 HTTP 프로토콜을 사용한다.(HTTP 버전은 1.1이상, 반드시 GET 메서드 사용)
- Host: 웹 서버 소켓의 주소
- Upgrade: 현재 클라이언트, 서버, 전송 프로토콜 연결에서 다른 프로토콜로 업그레이드(변경)하기 위한 규약
- Connection: Upgrade 헤더 필드가 명시되어 있는 경우, Upgrade 옵션이 지정되어 있는 Connection 헤더 필드도 함께 전송해야 한다.
- Sec-WebSocket-Key: 클라이언트와 서버가 서로를 구분할때 사용
- Origin: 클라이언트의 주소
- Sec-WebSocket-Protocol: 클라이언트가 요청하는 여러 서브 프로토콜을 의미(요청을 받으면 서버에서 지원하는 프로토콜이 무엇인지 반환해준다.)

### 응답 메시지
```bash
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbk+xOo=
```
- 101 Switching Protocols: 웹소켓이 연결되었다는 의미
- Sec-WebSocket-Accept: 클라이언트로 부터 받은 Sec-WebSocket-key를 사용하여 계산된 값(값이 일치하지 않으면 연결 x)

## 웹 소켓 동작 방법(데이터 전송)
- 핸드 쉐이크 완료 이후 동작
- 핸드 쉐이크 과정에서 사용되던 HTTP 프로토콜이 ws 또는 wss(https처럼 보안을 위한 SSL이 적용된 프로토콜) 로 전환된다.
- Message(여러 frame이 모여서 구성하는 하나의 논리적 메세지 단위)단위를 사용
- frame: communication에서 사용되는 가장 작은 단위의 데이터, 작은 헤더 + payload로 구성

## 웹 소켓 프로토콜 특징
- 최초 접속에서만 HTTP 프로토콜 위에서 handshaking을 하기 때문에 http header 사용
- 별도의 포트가 존재하지 않으며, 기존 포트를 사용(80, 443)
- 프레임으로 구성된 메시지를 사용해 송수신
- 메시지에 포할 포함될 수 있는 교환 가능한 메시지는 텍스트와 바이너리

## 웹 소켓의 한계
- HTML5 이전의 기술로 구현된 서비스에서는 별도의 기술(Socket.io, SockJS)을 사용해야 한다.
- 웹 소켓은 문자열을 주고 받을 수 있게 해줄뿐이다.
	- 문자열의 해석은 온전히 애플리케이션이 담당
- 형식이 정해져 있지 않아 애플리케이션에서 해석이 어렵다.
	- 이 때문에 sub-protocols을 사용해서 주고 받는 메시지의 형태를 약속하는 경우가 많다.
	- STOMP(Simple Text Oriented Message Protocol): sub-protoclo로 많이 사용되는 프로토콜

# REFERENCE
[[10분 테코톡] 🧲코일의 Web Socket](https://www.youtube.com/watch?v=MPQHvwPxDUw)

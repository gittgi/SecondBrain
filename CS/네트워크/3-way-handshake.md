---
tags:
  - 네트워크
  - CS_지식의_정석
  - TCP/IP
---
# 3-way-handshake

```table-of-contents
```

##  3-way-handshake

- [전송 계층](전송%20계층.md)의 TCP 통신에서 연결을 성립시키기 위한 과정이 3 웨이 핸드셰이크
- 총 3단계의 과정을 거침
	1. SYN 단계 : 클라이언트는 ISN[^1]을 담아서 서버측으로 SYN을 보냄
	2. SYN + ACK 단계 : 서버는 클라이언트의 SYN[^2]을 수신한 뒤, 서버의 ISN과 방금 수신한 클라이언트의 ISN에 +1 한 값을 승인번호(ACK[^3])로 클라이언트에게 전송
	3. ACK 단계 : 클라이언트는 서버의 ISN + 1 한 값을 승인번호로 다시 서버에 전송

## 클라이언트와 서버 상태
- 3 웨이 핸드셰이크 과정에 따라 클라리언트와 서버의 상태(state)가 변경됨
	- 처음엔 서버와 클라이언트 모두 **CLOSED**
		- 서버는 **LISTEN** 상태가 되어야 클라이언트의 요청을 받을 수 있음
	- 3 웨이 핸드셰이크가 시작되면 클라이언트는 서버 측으로 SYN을 전송하고 **SYN-SENT** 상태가 되어 ACK를 기다림
	- **LISTEN** 상태의 서버는 SYN을 받으면 **SYN-RECEIVED** 상태가 되고, SYN + ACK를 클라이언트에게 보냄
	- **SYN-SENT** 상태의 클라이언트는 SYN + ACK를 받으면 **ESTABLISHED** 상태가 되고, ACK를 다시 서버에 전송
	- **SYN-RECEIVED** 상태의 서버는 ACK를 클라이언트에서 받고, **RECEIVED** 상태가 됨

[^1]: ISN : TCP(Transfer Control Protocol) 기반 데이터 통신에서 각각의 새 연결에 할당된 고유한 32비트 시퀀스 번호. TCP 연결을 통해 전송되는 다른 데이터 바이트와 충돌되지 않도록 구분해주는 역할
[^2]: SYN : synchronization, 연결 요청 플래그
[^3]: ACK : acknowledgement, 응답 플래그

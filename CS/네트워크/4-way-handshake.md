---
tags:
  - 네트워크
  - CS_지식의_정석
  - TCP/IP
---
# 4-way-handshake

```table-of-contents
```

##  4-way-handshake

- [전송 계층](전송%20계층.md)의 TCP 연결을 성립시킬 때 [3-way-handshake](3-way-handshake.md) 과정이 필요하다면, 해제할 때는 4 웨이 핸드셰이크 과정이 필요
- 과정
	1. ESTABLISHED 상태의 클라이언트가 연결을 닫고자 할 때, FIN으로 설정된 세그먼트를 서버로 전송하고 FIN_WAIT_1 상태로 서버의 응답을 기다림
	2. ESTABLISHED 상태의 서버가 FIN으로 설정된 세그먼트를  받으면 클라이언트로 ACK라는 승인 세그먼트를 보내고 CLOSE_WAIT 상태로 진입, 서버 어플리케이션이 Close하도록 전달
	3. 클라이언트가 ACK를 받으면, FIN_WAIT_2 상태가 되어 서버가 FIN을 보낼때까지 대기
	4. 어플리케이션이 Close할 준비가 되면 클라이언트에 FIN을 전송하고 LAST_ACK 상태로 진입하여 클라이언트의 ACK를 기다림
	5. FIN_WAIT_2 상태의 클라이언트가 FIN을 받으면 서버 측에 ACK를 보내고 TIME_WAIT 상태로 진입하여 일정시간[^1] 동안 대기
	6. LAST_ACK 상태의 서버는 ACK를 받으면 종료 (CLOSED)
	7. 일정 시간이 지나면 클라이언트도 종료

## TIME_WAIT

- 클라이언트에서 TIME_WAIT 이후에 종료하는 이유
	1. 지연 패킷 등이 발생하여 아직 서버로부터 도착하지 않은 데이터 등을 기다리는 것으로 인한 데이터 무결성을 해결
	2. 서버가 LAST_ACK 상태가 아닌 CLOSED 상태로 바뀌어 있을 수 있도록 충분히 기다려주는 것 (CLOSED 상태가 아니면 다음 연결 때 오류 발생)

[^1]: TIME_WAIT : 보통 최대 패킷 수명(MSL, maximum segment lifetime)의 두배로 설정되어 있고, 이는 OS 마다 다름(Ubuntu, CentOS6는 60초, 윈도우는 4분)
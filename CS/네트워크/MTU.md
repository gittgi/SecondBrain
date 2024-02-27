---
tags:
  - 네트워크
  - CS_지식의_정석
aliases:
  - MSS
  - PMTUD
---
# MTU

```table-of-contents
```

##  MTU란

- Maximum Transmission Unit
- 네트워크에 연결된 장치가 받아들일 수 있는 최대 데이터 패킷의 크기
- 이 크기를 기준으로 데이터가 쪼개져서 패킷화
- 네트워크 경로상의 일부 장치의 MTU보다 패킷이 크면, 해당 패킷은 해당 장치로 넘어가기 전 장치에서 분할될 수 있음
- 단, 분할되지 않고 아예 전달하지 않는 경우가 있음
	- IPv6는 분할을 허용하지 않음
	- IPv4의 경우 IPv4헤더에 있는 Flags 비트가 1인 경우 (Don't Fragment) 분할하지 않음


## MSS란

- Maximum Segment Size
- payload의 크기만을 가리킴
- MTU == MSS + IP헤더 + TCP헤더
- 일반적으로 MTU의 경우 1500바이트 이지만, IP헤더(20바이트), TCP헤더(20바이트) 등을 고려해서 데이터 자체는 1460바이트 이하로 보내야 전달이 됨
	- TCP를 사용하지 않는 등의 경우에는 달라짐


## PMTUD

- Path MTU Discovery
- 수신자와 송신자의 경로 상에서 장치가 패킷을 누락한 경우, 테스트 패킷의 크기를 낮추면서 MTU에 맞게끔 반복해서 보내는 과정




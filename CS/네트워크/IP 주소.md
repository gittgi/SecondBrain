---
tags:
  - 네트워크
  - CS_지식의_정석
aliases:
  - IPv4
  - IPv6
---
# IP 주소

```table-of-contents
```

##  IP 주소

### IPv4

- 32비트로 표현되는 주소체계 (2^32 == 41억 9000만개의 주소)
- 8비트 단위로 점을 찍어 4부분으로 구분
- 8비트 2진수를 10진수로 변환시켜 표현
- 해당 주소체계만으로는 부족하기 때문에, NAT, 서브네팅 등의 부수적인 기술들 활용
- 예시 : 172.0.0.1


### IPv6

- 128비트로 표현되는 주소체계 (2^128 개의 주소)
- 많은 주소처리가 가능하기 때문에 NAT, 서브네팅 불필요
- 16비트씩 8부분으로 구분
- 16비트는 16진수로 변환되어 콜론으로 구분
- 연속되는 0의 경우에는 생략 될 수 있음
- 예시 :
	- 2001:0DB8:AC10:FE01:0000:0000:0000:0000
		- 2001:0DB8:AC10:FE01:: (0 생략됨)
- 앞의 64비트는 네트워크 주소를 뜻하고, 뒤의 64비트는 인터페이스(호스트) 주소를 뜻함
- IPSec[^1]이 내장되어 있음
- IPv4의 불필요한 헤더 필드를 제거해서 보다 빠른 처리
	- 체크섬 필드 삭제
		- IPv4의 경우 CRC[^2]를 통해 손상된 패킷을 확인하고 폐기하는 체크섬 필드 존재
		- IPv6의 경우 상위 프로토콜(TCP, UDP)에 체크섬 필드가 있기 때문에 효율화를 위해 삭제
		- 단 IPv6 + UDP 의 경우에는 UDP 헤더의 체크섬 필드를 사용한다고 설정해야 함[^3]
	- IPv4의 경우 헤더 길이가 가변적, IPv6의 경우 40바이트 고정이기 때문에 인터넷 헤더길이에 대한 정보 삭제
	- 이외 여러 불필요한 정보 삭제
- IPv4의 TTL[^4] -> IPv6의 HOP limit으로 용어변경
- IPv6의 경우 더 많은 주소를 표현하고 또 불필요한 헤더 삭제로 빠르며, IPSec 내장으로 보안성도 강화됨
	- 그러나 IPv6의 패킷 크기로 인해 일부 사용사례에서는 IPv4가 더 빠를 수 있음



## IP 주소 체계

- IP 주소는 네트워크 주소 / 호스트 주소 의 두 부분으로 나뉨
	- 네트워크 주소는 호스트들을 모은 네트워크를 지칭
	- 호스트 주소는 동일 네트워크(로컬 네트워크) 내의 호스트 들을 구분하기 위한 주소
		- 네트워크 호스트는 네트워크에 연결된 컴퓨터 및 기타 장치

### Classful IP Addressing

- 네트워크의 크기를 다르게 구분하여 클래스를 할당하는 주소체계
- 클래스는 A부터 E까지 5개로 구분됨

#### 클래스 A
- IPv4의 맨 앞 비트가 0인 주소들이 클래스 A 주소 범위
	- 00000000.00000000.00000000.00000000 부터 01111111.11111111.11111111.11111111 까지
	- 즉, 맨 앞 부분이 1 ~ 126인 주소들
		- 127.X는 루프백 주소[^5], 0.0.0.0은 알수 없는 대상에 달아두는 임시주소라 예외
- 2^24 - 2 = 하나의 네트워크 주소당 약 1600만개의 호스트 주소 보유

#### 클래스 B
- IPv4의 두번째 비트가 0인 주소들이 클래스 B 주소 범위
	- 10000000.00000000.00000000.00000000 부터 10111111.11111111.11111111.11111111 까지
	- 맨 앞 부분이 128 ~ 191인 주소들
- 2^16 - 2 = 하나의 네트워크 주소당 약 6만 5000개의 호스트 보유

#### 클래스 C
- IPv4의 세번째 비트가 0인 주소들이 클래스 C 주소 범위
	- 11000000.00000000.00000000.00000000 부터 11011111.11111111.11111111.11111111 까지
	- 맨 앞 부분이 192 ~ 223인 주소들
- 2^8 - 2 = 하나의 네트워크 주소당 254개의 호스트 보유

#### 이외 클래스
- 클래스 D의 경우 [멀티캐스트](유니캐스트,%20멀티캐스트,%20브로드캐스트.md)용 주소
- 클래스 E의 경우 예비용(Reserved) 주소


> [!Important] 항상 주소 2개가 빠지는 이유
> - 맨 앞자리는 네트워크 주소, 마지막 주소는 브로드캐스팅 주소로 남겨둠

#### 클래스풀의 문제점
- 필요한 네트워크 크기에 따라서 너무 많은 네트워크 주소가 필요하거나, 낭비되는 호스트 주소들이 많을 수 있음
	- 5000개의 주소가 필요한 경우:
		- 클래스 C(254) 네트워크의 경우 너무 많은 네트워크 주소가 필요
		- 클래스 B(65534) 네트워크의 경우 너무 많은 호스트 주소가 낭비됨

### Classless IP Addressing

- 위와 같은 클래스풀의 문제점을 해결하기 위해 등장, 현재 사용되고 있는 주소체계
- 클래스를 나누는 대신, **서브넷마스크**를 활용하여 네트워크 주소와 호스트 주소를 나눔
	- 서브네팅 : 네트워크를 나눈다는 뜻
	- 서브넷 : 서브 네트워크, 나뉜 네트워크
	- 서브넷마스크 : 서브네트워크를 위한 비트마스크


#### 서브넷마스크
- 네트워크의 주소부분은 1, 호스트 주소부분은 0 으로 표시
	- 예시 : 11111111.11111111.11111111.11000000
- IP주소와 서브넷마스크를 & 연산 하게 되면, 네트워크 부분만 남아서 네트워크 구분 가능
- 앞에서 부터 1인 비트의 개수를 /n 의 형태로 표현
	- 123.12.12.12/28 -> `/28`이면 11111111.11111111.11111111.11110000의 서브넷마스크
- 서브넷마스크를 조절하는 것으로, 네트워크 주소에 포함되는 호스트 주소의 수를 조절 가능
	- 123.12.12.12/28 -> 16 - 2의 호스트 주소 확보 가능
	- **이 덕분에 낭비되는 호스트 주소를 줄일 수 있음**


## public IP와 private IP

- IP 주소의 부족을 해결하기 위해 공인 IP 와 사설 IP로 나누고 NAT[^6] 기술을 도입
- 공유기 등의 라우터가 하나의 공인 IP 주소를 부여받고, 이 라우터를 통해 접속하는 개별 기기들은 사설 IP를 부여 받음
- 개별 기기들이 인터넷으로 통신을 할 경우, 라우터의 공인 IP로 통신을 하게되고, 라우터에 데이터가 도착하면 NAT를 통해 사설 IP에 해당하는 기기로 응답됨
- 이 NAT를 통해 내부 네트워크 IP가 노출되지 않아 안전성이 제고되는 효과가 있음







[^1]: IPSec : 데이터 패킷을 암호화하는 네트워크 프로토콜 제품군
[^2]: CRC (Cyclic Redundancy Check): 순환중복검사. 주어진 데이터 값에 따라 CRC 값을 계산하여 데이터에 붙여서 전송. 데이터 받은 값에서 CRC 값을 계산하여 원래 값과 비교. 차이가 있다면 전송 과정에서 오류가 있었던 것
[^3]: TCP 통신의 경우 체크섬 필드가 필수이지만, UDP 통신의 경우 체크섬 필드를 0으로 설정하면 수신측에서는 체크섬 검증을 진행하지 않음(비활성)
[^4]: TTL (Time To Live): 패킷이 네트워크에서 무한 순환하는 것을 방지하기 위해 패킷이 네트워크에서 라우터를 거칠때마다 TTL 값이 1씩 감소. 0이 되면 패킷 폐기.
[^5]: 루프백 주소 : 자기 자신을 가리키기 위한 주소(localhost). 보통 127.0.0.1을 사용
[^6]: Network Address Translation : 패킷이 트래픽 라우팅 장치를 통해 전송되는 동안 패킷의 IP 주소를 변경, IP 주소를 다른 IP 주소로 매핑하는 방법


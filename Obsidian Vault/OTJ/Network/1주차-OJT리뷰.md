# 1. Git
---
## 1.1 commit
---
`기존` 커밋 남발 (이미지 경로 수정)

`변경` 커밋할 때는 신중하게, 옵시디언 전용 마크다운 기능과 구별 필요

## 1.2 numbering
---
`기존` '#'으로 구분은 했으나 넘버링 없음

`변경` 넘버링 추가 (조범준님 정리 방식 참고)

# 2. Network
---
## 2.1 window, mss, mtu 
---
### 2.1.1 window size
- 정의
	- 송신자(sender)가 보낼 수 있는 최대 데이터 크기(= 수신자가 현재 받을 수 있는 데이터 양)
		- 이때 데이터는 PDU 단위의 데이터(=헤더를 제외한 세그먼트 payload 부분)
	- **수신자가 버퍼에 담을 수 있는 최대 공간**
	- 수신 측이 ACK 없이 한꺼번에 수신할 수 있는 데이터의 총량(바이트 단위)
- 목적
	- 상황에 따라 데이터가 소비되는 속도가 다르기 때문에 빈 공간은 때에 따라 다름. 따라서 **수신자(reciever)가 송신자(sender)에게** 계속 window를 알려줘야함
	- **흐름 제어**에 사용됨
- 특징
	- 대부분의 프로그램들은 클라이언트가 서버보다 데이터를 주기보단 받는 것이 더 많기 때문에 클라이언트가 서버보다 더 많은 window size값을 가짐
	- 연결 설정(handshaking) 중에 협상됨
		- SYN(client): 자신의 window와 필요한 경우 window scale option을 포함해서 보냄
		- SYN&ACK(server): 자신의 window와 window scale option을 포함해 전송
		- ACK(client): 초기 window 값 확정
- window size 결정 방식
	- 네트워크 상황(혼잡도)과 수신자의 버퍼 크기 두 가지를 고려
		- **혼잡 제어(CWND, Congestion Window or flight size)**
			- 네트워크 혼잡 상태를 파악하여 전송량을 조절
			- **송신자(sender)가 ack를 받기 전에 보낼 수 있는 패킷양**
			- 라우터의 버퍼가 유한하기 때문에 혼잡 발생. 이로 인해 패킷 delay나 loss가 일어날 수 있기 때문에 congestion control이 필요
			- 라우터에 들어가는 패킷의 양이 증가함에 따라 나가는 패킷의 양을 그래프로 나타냈을 때 해당 그래프에서 최대값 지점을 찾는 과정이 congestion control이라고 보면됨
			- AIMD(Additive Increase & Multiplicative Decrease): 패킷 loss가 발생하지 않는다면 점진적으로 cwnd값을 증가 시킴. 만약 패킷 loss가 발생시 cwnd값을 반으로 확 줄임 (처음부터 너무 천천히 증가)
			- TCP slow start: AIMD 방식을 보완하기 위한 방식. 처음 cwnd값을 1mss로 시작하여 매 rtt마다 cwnd값을 두 배로 증가시킴 -> 언제까지? -> cwnd값이 ssthresh(slow start threshold)에 도달할 때까지
			- congestion avoidance: cwnd값이 ssthresh에 도달한 후 congestion avoidance과정에 돌입. 해당 과정은 매 rtt마다 cwnd 값을 cwnd + mss*(mss/cwnd)로 증가시킴 -> 매 rtt마다 cwnd값을 1 증가
			- loss 발생시 tcp는 두가지 유형으로 나누어 대응함
			- 먼저 중복된 ack들로 인한 loss 판단이 있음. 중복된 ack들이 계속해서 온다는 것은 특정 패킷만 loss되었고 나머지 패킷들은 제대로 전송되었음을 의미. -> cwnd를 loss가 발생했을 때의 절반으로 줄이고 다시 congestion avoidance 과정 실행
			- 두번째는 timeout으로 인한 loss 판단임. 전송한 모든 패킷들이 loss되었음을 의미 -> cwnd를 1로 줄이고 ssthresh값도 기존의 절반으로 줄임. 이후 다시 slow start 과정 시행 
			- TCP Tahoe 방식은 loss 발생시 무조건 두번째 방식으로 처리
			- TCP Reno 방식은 loss 발생시 첫/두번째 방식 모두 사용하여 처리
			- TCP congestion control의 핵심 -> cwnd 값을 잘 조절하는 것
		- 흐름 제어(RWND, Receiver Window)
			- 수신측 버퍼의 크기에 따라 송신 속도를 조절
	- 순서
		- 1. 송신자는 cwnd, rwnd를 모두확인
		- 2. 실제 전송할 수 있는 window size는 **min(cwnd, rwnd)** 로 결정됨
		- 3. 위의 과정은 네트워크 상태와 수신자의 상태에 따라 동적으로 계속해서 조절
	- 일반적으로 rwnd > cwnd 이므로 보통 송신자의 window size는 cwnd에 의해 결정됨
	- 따라서 단위시간당 TCP가 보낼 수 있는 데이터의 양의 대략 cwnd/rtt로 계산
- 확장
	- tcp header의 크기는 2bytes(16 bits) -> 수신 버퍼의 여유 용량이 65,535 bytes
	- 요즘 인터넷 속도에 비해 부족한 패킷 크기 
	- RFC 1323 - tcp window scale option
		- 고성능을 위한 tcp 확장(TCP Extensions for High Performance)
		- tcp window size를 늘릴 수 있는 기능 == **TCP Window Scaling**
		- 3 way hanshake 과정에서만 window 값과 함께 scale 값을 같이 주고 이 값들은 연결 내내 지속됨
		- ![[Pasted image 20251125095251.png]]
		- tcp.option_kind: tcp 옵션을 선택 (window는 type 3임)
			- 더 많은 타입 넘버와 매핑되는 옵션을 확인하고 싶다면 [TCP 옵션](http://www.ktword.co.kr/test/view/view.php?no=1031) 참고 또는 RFC 1323에서도 명시되어있음
		- 필터링된 결과값에 SYN 패킷들만 있는 것을 볼 수 있음
			- window 값은 핸드쉐이킹 중 SYN 패킷에서 클라이언트와 서버가 서로 자신의 window값(+window scale option)을 전송하기 때문
		- ![[Pasted image 20251125130537.png]]
		- Shift count: 추가적으로 더 늘릴 값
			- 잡힌 패킷에서 Shift count가 7로 설정됨 -> window값에 2^7을 곱함 -> window size * 2^shift count
			- shift count의 값은 14이하이어야함 (14초과시 14로 사용)

### 2.1.2 mss
---
- 정의
	- 하나의 tcp segment가 담을 수 있는 최대 데이터(payload) 크기
- 목적
	- 네트워크 경로에서 한 번에 전송될 수 있는 개별 패킷의 물리적인 최대 크기를 제한하여 효율적이고 안정적인 전송 보장
- 특징
- mss 결정 방식
	- MTU를 기반으로 계산: MSS = MTU - IP header - TCP header
	- IP와 TCP의 헤더 길이는 거의 항상 **20 바이트**

### 2.1.3 mtu
---
- 정의
	- 가장 큰 데이터 패킷을 나타내는 측정값
	- 단일 패킷으로 전송할 수 있는 최대 데이터 패킷 크기
- 목적
- 특징
	- MTU를 초과하는 데이터 패킷은 더 작은 조각으로 분할됨(**단편화**)
		- 분할을 허용하지 않는 경우
			- IPv6를 사용하는 경우
			- IP header에 Don't Fragment 플래그 활성화 되어있는 경우
	- 네트워크 연결에 사용되는 장치(라우터/스위치 등)가 한 번에 처리할 수 있는 최대 데이터 패킷 크기
	- 가장 일반적인 이더넷의 MTU 값은 1500 바이트
- mtu 결정 방식
	- 전송 매체별에 따라 상이함

| 전송 매체       | MTU       |
| ----------- | --------- |
| IPv4        | 64 ~      |
| IPv6        | 1280~     |
| Ethernet v2 | 1500      |
| Jumbo Frame | 1501-9216 |
| WLAN        | 7981      |

- 확장
	- 왜 일반적으로 이더넷 v2를 사용하는가
		- 참고 링크1 : [How 1500 bytes became the MTU of the internet](https://blog.benjojo.co.uk/post/why-is-ethernet-mtu-1500)
		- 참고 링크2 : [MTU, 진짜 1500 Byte 일까?](https://se-juno.tistory.com/2)
		- 이더넷이 구현이 단순하고 비용이 저렴해서 빠르게 표준으로 잡음
	- 이더넷 i 과 ii 차이
		- i은 과거 IBM/Novell 등 일부 전용망에서 사용하고 ii은 IPv4/6, ARP 사실상 전세계 인터넷 표준으로 사용되고 잇음
		- 헤더 구조에서 i는 EtherType이 아닌 Length(mac 데이터 필드 길이)를 사용 반면 ii는 EtherType을 통해 상위 프로토콜을 직접 지정할 수 있음
	- wireshark에서 더 큰 길이의 mtu 값
		- MTU(1500bytes) 비효율성
			- 10기가 속도로 데이터를 보낼 때 1500바이트 패킷을 초당 약 80만 개 만들어서 보내야됨
			- CPU는 순수 데이터 처리보다 패킷 관리 작업에 CPU 자원을 불필요하게 많이 씀 (CPU 오버헤드)
			- 해결방법
				- 1. MTU 늘리기 
				  점보 프레임: MTU를 크게 늘리는 기술. 
				  한계점 : 네트워크 장비들도 점보 프레임을 처리할 수 있도록 설정되어야함. 하지만 일반적인 ISP는 MTU 1500바이트를 표준으로 사용함 -> 현실적으로 의미 없음
				- 2. NIC에게 넘기기
					- 1과 같이 외부 인터넷 연결의 MTU를 바꾸기 어려우니까 컴퓨터 안에서 오버헤드를 줄여야함
					  이 역할을 Offload 기능이 담당함 (ex: TSO, GSO, LRO)
					  **CPU가 할일을 NIC에게 넘겨 CPU의 부담을 덜어주는 기술**
					- Offload: 컴퓨터 시스템 내에서 특정 작업의 처리 부담을 주 CPU에서 전용 하드웨어 칩이나 특수 코어로 이관하는 기술
		- TOE(TCP Offload Engine)
			- NIC에서 사용되는 기술로 보통 OS에서 처리되는 tcp/ip 스택을 nic로 내려서 처리하는 것 (네트워크 처리가 크게 필요한 고속네트워크에서 유용하게 사용됨)
			- TSO(TCP Segmentation Offload)
				- 컴퓨터에서 **밖으로 나가는** 패킷의 CPU 오버헤드 감소
				- CPU가 큰 데이터 덩어리를 NIC 에게 넘기면 NIC에서 작은 패킷으로 쪼갬
			- GSO(Generic Segmentation Offload)
				- TSO 확장 - tcp외의 다른 프로토콜에도 적용
			- LRO(LargeReceive Offload)
				- **외부에서 들어오는** 패킷의 CPU 오버헤드 감소
				- NIC가 작은 패킷들을 최대한 합쳐서 데이터 덩어리로 만들어 CPU에게 전달

### 2.1.4 window, mss, mtu 관계 정리
---
- window와 mss 구분
	- MSS는 **개별 세그먼트의 최대 크기**를 제한하는 반면, Window Size는 **전체적인 데이터 흐름의 양**을 조절
- MTU와 MSS의 주요 차이점
	- 패킷이 장치의 MTU를 초과하면 해당 패킷이 더 작은 조각으로 분할(분편화)됨
	- 패킷이 MSS를 초과하면 삭제되고 전달되지 않음

## 2.2 웹 소켓과 소켓의 차이점
---
- TCP/IP Socket
	- 정의
		- 양 끝단이 서로 네트워크 통신할 수 있도록 하는 인터페이스로 ip 주소와 포트 번호의 조합을 통해 데이터를 주고받는 소프트웨어적 개념/인터페이스
	- 목적
		- 네트워크 통신 엔드포인트
			- IP주소와 포트 번호 결합하여 네트워크 상의 특정 호스트의 특정 프로세스를 고유하게 식별하고 연결하는 기능
		- 유저 영역과 커널 영역간의 매개체
			- 메모리는 프로세스가 실행되는 유저 영역과 하드웨어에 명령을 전달하는 커널 영역으로 구분되어있음. 사용자가 프로세스를 통해 어떤 기능을 사용했을 때 그에 매핑되는 명령을 커널을 통해 하드웨어로 명령을 전달하기 위해서 그 중간에 위치해 중간 매개체 역할을 수행함
	- 계층: 4계층
	- 데이터 전송 방법
		- 바이트 스트림을 통한 데이터 전송 사용
- Web Socket
	- 정의
		- http/s 프로토콜 위에 실시간 양방향 통신 기능을 추가한 프로토콜
	- 계층: 7계층
	- 데이터 전송 방법
		- 구조화된 메시지 형식의 데이터를 다룸
	- 특징
		- 초기 핸드셰이크 이후 연결이 유지되어 데이터를 지속적으로 주고 받음
		- 연결 후 헤더 정보가 매우 작아 오버헤드가 적고 빠름

## 2.3 비연결성/무상태성
---
- 비연결성(Stateless/Connectionless)
	- 정의
		- 각 요청-응답마다 연결을 맺고 끊는 방식
	- 예시
		- 브라우저가 서버에 html 문서를 요청하고 응답 받은 후 연결을 끊음
		- 브라우저가 html 분석해보니 페이지 로드를 위해 css, js, 이미지 파일 등 추가 리소스가 필요함
		- 그럼 브라우저는 서버와 새로운 연결을 맺고 (tcp 3 way handshake)데이터를 받고 연결을 끊음 (반복)
	- 장점
		- 서버는 수 많은 클라이언트와의 연결을 계속 유지할 필요가 없어 서버 자원을 효율적으로 관리 가능
	- 보완
		- http/1.1 이후로는 지속 연결 기능이 기본적으로 적용되어 일정 시간 동안 연결을 유지할 수 있게 됨
- 무상태성
	- 정의
		- 서버가 클라이언트의 이전 요청이나 현재 상태에 대해 전혀 기억하지 않음
	- 예시
		- 클라이언트가 (1) 로그인하고 (2) 장바구니에 물건을 담았다면 서버는 (1),(2) 요청이 동일한 사용자로부터 온 건지 기본적으로 알 수 없음
	- 장점
		- 서버가 상태를 저장할 필요가 없어 확장성이 매우 높아짐
	- 보완
		- 실제 웹 서비스에서는 로그인 유지, 장바구니 정보를 저장해야 하므로 쿠키, 세션, 토큰 등의 기술을 사용하여 상태 정보를 관리함

## 2.4 TCP flags
---
- 역할 및 특징
	- 패킷의 목적과 상태를 알림
	- 6개의 1비트로 구성됨
	- tcp 연결 수립,데이터 전송, 종료 등 전반적인 과정에 걸쳐 사용됨
- 종류
	- SYN(SYNchronization)
		- 연결 요청. 통신 시작 시 세션을 연결하기 위한 플래그
		- 여기서 말한 세션은 TCP 연결 그 자체를 의미(신뢰성 있는 통신 채널 수립)
	- ACK(ACKnowledgement)
		- 응답 플래그.  송신측으로부터 패킷을 잘 받았다는 걸 알려주기 위한 플래그
	- FIN(FINish)
		- 연결 종료 플래그. 더 이상 전송할 데이터가 없고 세션 연결을 종료 시키겠다는 플래그
	- RST(ReSeT)
		- 연결 재설정 플래그. 비정상적인 상황에서 연결을 강제 종료하거나 재설정할 때 사용됨
	- PSH(PuSH)
		- 버퍼가 채워지기를 기다리지 않고 받는 즉시 전달
		- 버퍼링 없이 응용프로그램에게 바로 전달
	- URG(URGent)
		- 해당 패킷이 긴급 데이터를 포함하고 있음을 알림

## 2.5 traceroute 작동 방식
---
### 2.5.1 ICMP
---
> traceroute는 ICMP error 메세지를 통해 진행됨. 따라서 traceroute 작동 방식을 알기전에 ICMP 원리를 알고 가야함

- 정의
	- 오류 보고나 네트워크 진단을 위한 메시지를 주고받는 프로토콜
- 원리
	- IP를 통한 메시지 전달: IP 프로토콜 위에서 작동하며 IP 패킷 전송 상태에 대한 정보를 전달함
	- 메시지 유형을 나타내는 타입(Type)과 해당 타입의 세부 내용을 나타내는 코드(Code)로 메시지 구분
		- Type 0: Echo reply - Echo 메세지 응답 (ping 응답)
		- Type 3: Destination unreachable - 목적지 도달 불가
		- Type 4: Source quench - Congestion control 의 용도
		- Type 5: Redirect - 더 빠른 경로가 있다고 알려줌
		- Type 8: Echo request - echo 메세지의 요청 (ping 요청)
		- Type 11: Time exceeded - TTL 초과
	- 에코 요청/응답 (ping): 목적지 호스트가 응답하는지 확인하기 위해 사용하는 메시지

### 2.5.2 traceroute
---
- 사용목적
	- 특정 IP까지의 라우팅 경로를 알고 싶을 때 사용하는 명령어
- 방식
	1. TTL을 1으로 패킷을 전송함
	2. TLL이 0이 된 바로 다음 라우터는 ICMP error 메세지를 돌려줌 (경로에 있는 첫 번째 라우터 IP 알아냄)
	3. TTL을 2로 패킷 전송 
	4. TTL이 0이된 경로중 2번째에 있는 라우터는 ICMP error 메세지를 돌려줌 (경로에 있는 두번째 라우터 IP 알아냄)
	5. 위와 같은 방식으로 도착할 때 까지 모든 TTL을 늘려가며 전송함
	6. 목적지의 ICMP 메세지를 받으면 라우팅 경로의 모든 IP를 알 수 있음
- 참고링크 : [What is Traceroute?](https://www.checkpoint.com/kr/cyber-hub/network-security/what-is-traceroute/)

## 2.7 4계층 프로토콜 - QUIC
---
- 정의
	- QUIC(Quick UDP Internet Connection)
	- UDP 기반의 차세대 전송 계층 프로토콜 (구글에서 개발)
	- TCP와 TLS의 단점을 보완하여 더 빠르고 안전한 통신을 가능하게함
- 특징
	- UDP 기반으로 연결 수립 시간을 단축하고 재연결 시 핸드셰이크 과정 생략
	- 고유 식별자인 커넥션 ID를 사용하여 IP 주소나 포트가 변경되어도 연결을 유지할 수 있음
	- TLS 1.3을 기반으로 한 암호화 기능이 내장되어있음
- 참고 링크1 : [QUIC - Wikipedia](https://en.wikipedia.org/wiki/QUIC)
- 참고 링크2 : https://dl.acm.org/doi/pdf/10.1145/3098822.3098842

# 3. 이해도 점검
## 3.1 Window Size, CWND, RWND 의 관계
---
### 3.1.1 질문
---
**TCP 통신에서 송신자가 실제로 전송할 수 있는 최대 데이터 양을 결정하는 최종적인 기준은 무엇이며, 이 기준이 흐름 제어(Flow Control)와 혼잡 제어(Congestion Control)에 각각 어떻게 기여하는지 설명해 주세요.**

- 특히, "일반적으로 rwnd > cwnd 이므로 보통 송신자의 window size는 cwnd에 의해 결정된다"는 문장이 시사하는 바를 **네트워크 관점**에서 설명해 주세요.

### 3.1.2 대답
---
tcp 통신에서 송신자가 실제로 전송할 수 있는 최대 데이터 양(Window)은 CWND와 RWND 중 최소값으로 결정한다. 하지만 일반적으로 결정하는 최종적인 기준은 CWND(Congestion Window)이다.
여기서 RWND는 서버 또는 클라이언트 등의 호스트에서 수용할 수 있는 데이터의 최대 양 버퍼 양이고 CWND는 네트워크에서 혼잡으로 인해 패킷이 loss되는 상황을 피하기 위해서 보낼 수 있는 최대 버퍼 양이다.

### 3.1.3 피드백 및 정리
---
- RWND (흐름 제어): 수신자(엔드포인트)의 처리 능력에 중점을 둠
- CWND (혼잡 제어): 네트워크 경로의 혼잡 상태에 중점을 둠

- 일반적으로 엔드포인트(서버/클라이언트)의 버퍼 공간(RWND)은 네트워크 경로(CWND)보다 충분히 크도록 설계되어있음. 따라서 송신자는 수신자가 받아들일 준비는 되어 있지만 네트워크가 혼잡하여 중간에 패킷이 유실될 위험이 더 크다고 판단하고 네트워크 상황을 나타내는 CWND를 기준으로 전송량을 조절하게 됨

## 3.2 MSS와 MTU, 그리고 단편화(Fragmentation)
---
### 3.2.1 질문
---
**MTU, MSS, 그리고 IP 단편화(Fragmentation) 간의 관계를 설명해 주세요.**

- **MSS**의 목적이 IP 단편화를 **방지**하는 것과 어떤 관련이 있는지 설명하고, MSS 값이 MTUls  값보다 항상 작은 이유를 공식과 함께 설명해 주세요.

- IP 단편화가 발생했을 때 네트워크 성능에 미치는 **주요 단점**을 1~2가지 언급해 주세요.

### 3.2.2 응답
---
MSS는 segment의 최대 payload값 크기를 지정한 값이며 MTU는 MSS값에 IP, TCP 헤더가 붙은 값으로 MSS보다 항상 클 수 밖에 없다. MTU = IP, TCP header + MSS

IP 단편화가 발생하면 단편화된 패킷들에 순서 번호를 부여하고 단편화된 패킷들을 다시 순서에 알맞게 합쳐야하기 때문에 단편화가 많이 될 수록 오버헤드가 발생하여 전송 속도가 느려질 수 있고 자원을 과도하게 사용할 수 있다.

### 3.2.3 피드백 및 정리
---
- MTU = IP header + TCP header + MSS
- MSS: TCP Segment가 담을 수 있는 최대 페이로드 크기
- MTU: 패킷이 담을 수 있는 최대 크기

- 단편화의 단점
	- 오버헤드 발생(재조립 필요)
	- 단편화된 조각 중 하나라도 손실되면 전체 IP 패킷을 재조립할 수 없게 되어 모든 조각을 다시 전송해야 하는 비효율성 발생생

## 3.3 TCP Tahoe와 TCP Reno의 손실 복구 전략 차이
---
### 3.3.1 질문
---
**TCP Tahoe와 TCP Reno가 패킷 손실을 감지했을 때, 각각 어떤 전략을 취하는지, 특히 '중복된 ACK(Duplicate ACK)'를 받았을 경우 어떻게 다르게 대응하는지 설명해 주세요.**

- 이러한 차이가 '빠른 회복(Fast Recovery)' 개념과 어떻게 연결되는지 간단히 설명해 주세요.

### 3.3.2 응답
---
TCP Tahoe는 패킷 손실을 감지했을 때 cwnd를 1로 줄이고 매 rtt 마다 1 cwnd씩 증가한다. 하지만 Reno 방식은 중복된 ACK를 받았을 경우 cwnd를 절반으로 줄이고 ssthresh도 절반으로 줄인다음 ssthresh전까지 2배씩 증가하다가 ssthresh를 넘게 되면 매 rtt마다 1씩 증가하는 방식을 택한다. 하지만 timeout을 받았을 경우에는 tahoe 방식과 같은 방식으로 작동한다.

### 3.3.3 피드백 및 정리
---
- Tahoe(Timeout 감지): 손실 감지 시 무조건 CWND을 1로 줄이고 Slow Start(CWND 2배 증가) 시작 
- Reno (중복 ACK 감지)
	- Fast Retransmit: 세 번째 중복 ACK를 받으면 Timeout을 기다리지 않고 손실된 패킷을 바로 재전송
	- Fast Recovery: CWND를 절반으로 줄인 후 Slow Start가 아닌 Congestion Avoidance 단계로 바로 진입. 이를 통해 Slow Start의 느린 시작을 피하고 네트워크를 빠르게 회복 시킴.

## 3.4 웹 소켓과 HTTP/1.1의 비연결성 보완
---
### 3.4.1 질문
---
**웹 소켓(WebSocket)이 등장하기 이전에, HTTP/1.1은 비연결성(Connectionless)의 단점을 어떻게 보완하려고 했는지 (정리하신 내용에 근거하여) 설명해 주세요. 또한, 웹 소켓이 이러한 HTTP의 통신 방식과 근본적으로 다른 점은 무엇이며, 이로 인해 얻는 실시간 통신에서의 이점은 무엇인가요?**

### 3.4.2 응답
---
웹 소켓이 등장하기 이전에 http 1.1은 비연결성을 보완하기 위해서 conn: keep-alive 와 같은 http 헤더에 옵션을 선택할 수 있도록 하였다. 웹 소캣과 http 통신 방식의 근본적인 차이는 실시간 통신인데 웹 소캣은 한 번 연결하면 지속적으로 연결되며 실시간 통신에 유리하다.

### 3.4.3 피드백 및 정리
---
- http/1.1 보완
	- Connection: keep-alive를 사용해 일정 시간 동안 tcp 연결을 유지하고 여러 요청/응답을 처리하여 연결을 3 way handshake 오버헤드를 줄임
- 웹 소켓의 근본적 차이
	- 웹 소켓은 초기 http 연결을 upgrade한 후 tcp 위에서 양방향 통신 채널을 영구적으로 수립함. 
	- 서버와 클라이언트가 필요할 때 언제든지 데이터를 서로에게 push 가능.
	- 실시간성과 오버헤드 감소의 이점

## 3.5 QUIC 프로토콜의 주요 장점
---
### 3.5.1 질문
---
**QUIC 프로토콜이 TCP의 단점을 극복하고 더 빠르고 안전한 통신을 가능하게 하는 핵심적인 특징 3가지를 정리해 주세요.**

- 특히, '고유 식별자인 커넥션 ID'를 사용하는 것이 모바일 환경에서 IP 주소가 변경될 때 어떤 이점을 제공하는지 설명해 주세요.

### 3.5.2 응답
---
QUIC 프로토콜은 고유 식발자인 커넥션 ID를 사용하여 연결하고 있기 때문에 ip 주소가 변하는 등의 기존 환경이 변하여도 지속적으로 사용가능하다는 이점이 있다. 또한 QUIC 프로토콜은 http tls tcp 를 합쳐 handshaking과정을 축소화하고 udp를 기반으로 통신하여 빠른 통신을 할 수 있다는 이점이 있다. 

### 3.5.3 피드백 및 정리
---
- QUIC 이점
	- 커넥션 ID: IP주소나 포트가 변경되는 모바일 환경에서 기존 tcp 연결이 끊기는 문제를 해결하고 연결을 유지할 수 있게 함
	- UDP 기반 & handshake 축소: tls1.3을 내장하여 tcp 3 way handshake와 tls handshake를 결합함. tcp+tls 연결 수립을 1rtt로 단축하거나 재연결 시 0rtt로 즉시 데이터 전송 가능
	- Hol-Blocking 해결: 스트림 다중화(UDP 기반)로 구현하여 tcp의 HoL Blocking 문제를 근본적으로 해결함.
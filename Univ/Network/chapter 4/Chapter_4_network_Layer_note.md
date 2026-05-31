# Chapter 4 손필기 전사용 노트

## 목적

- 시험 직전 복습 가능
- 손으로 전사 가능
- 교안의 큰 흐름 + 개념 연결 + 시험 포인트 통합
- 애매한 표현을 줄이고, 왜 그런 설계가 필요한지까지 같이 정리

## 이 노트의 사용 방식

- 먼저 외울 핵심 문장부터 본다
- 그 다음 개념별 묶음으로 큰 흐름을 잡는다
- 각 개념에서
  - 정의
  - 왜 필요한가
  - 무엇과 비교되는가
  - ASCII 도식
  - 시험 포인트
  순서로 본다
- 마지막 비교 정리로 헷갈리는 개념을 정리한다

---

# 0. 먼저 외울 핵심 문장

- 네트워크 계층의 핵심은 **패킷을 목적지 쪽으로 보내는 것**이다.
- **Forwarding**은 한 라우터 안에서 입력 포트 → 출력 포트로 넘기는 동작이다.
- **Routing**은 출발지에서 목적지까지 어떤 경로로 갈지 정하는 전체 길찾기다.
- **Data plane**은 각 라우터의 로컬 동작이고, **control plane**은 네트워크 전체의 경로 논리다.
- 인터넷의 기본 서비스 모델은 **best effort**이다.
- Best effort는 **전달 성공, 순서, 지연, 대역폭을 보장하지 않는다.**
- 라우터는 들어온 패킷 헤더를 보고 **forwarding table**에 따라 내보낸다.
- 전통적 포워딩은 **destination IP address 기반**이고, SDN/OpenFlow는 더 많은 필드를 본다.
- **Longest prefix matching**은 여러 엔트리가 동시에 맞을 때 가장 구체적인 엔트리를 선택하는 규칙이다.
- 라우터 내부는 크게 **input port, switching fabric, output port, routing processor**로 본다.
- 입력 포트에서 막히는 대표 문제는 **HOL blocking**이다.
- 출력 포트는 패킷이 몰리면 **buffering, queueing, loss, scheduling 문제**가 생긴다.
- 스케줄링은 단순한 성능 문제가 아니라 **누가 더 좋은 서비스를 받는가**의 문제이기도 하다.
- IP 주소는 **호스트 전체가 아니라 인터페이스(interface)에 부여**된다.
- 같은 subnet에 있는 인터페이스끼리는 **중간 라우터 없이 직접 도달 가능**하다.
- CIDR은 **네트워크 부분 길이를 고정 클래스 없이 유연하게 자르는 방식**이다.
- 호스트는 보통 DHCP로 IP 주소를 받는다.
- DHCP의 기본 흐름은 **Discover → Offer → Request → ACK**다.
- NAT는 여러 사설 주소 장치가 **하나의 공인 IPv4 주소를 공유**하게 해준다.
- NAT는 주소 부족 완화에 유용하지만, **종단간 원칙(end-to-end argument)** 을 약하게 만든다.
- IPv6의 핵심은 **128비트 주소 공간 확대 + 헤더 단순화**다.
- IPv6는 라우터 처리 속도를 위해 **헤더 체크섬, 라우터 단편화 기능** 등을 없앴다.
- IPv4와 IPv6가 섞인 환경에서는 **tunneling**을 쓴다.
- Generalized forwarding은 패킷 헤더를 **match**하고, 그 결과로 **action**을 수행하는 방식이다.
- OpenFlow는 router, switch, firewall, NAT를 **하나의 match+action 관점**으로 묶어 본다.
- Middlebox는 단순 IP 라우팅 외의 기능을 수행하는 **중간 장치**다.
- 인터넷 철학의 핵심은 **IP라는 얇은 허리(thin waist)** 와 **복잡성은 네트워크 끝단에 둔다**는 생각이다.

---

# 1. 네트워크 계층의 큰 그림 (p4-2 ~ p4-12)

## 1-1. 정의 / 큰 그림

네트워크 계층은  
송신 호스트에서 수신 호스트까지  
패킷(datagram)을 전달하는 계층이다.

전송 계층은
- 프로세스 대 프로세스 통신

네트워크 계층은
- 호스트 대 호스트 전달

즉,
- 전송 계층이 "누구 프로그램에게 줄까?"라면
- 네트워크 계층은 "어느 컴퓨터 쪽으로 보낼까?"이다

교안 기준 핵심 질문은 2개다.

1. **Forwarding**
   - 지금 이 라우터에 들어온 패킷을
   - 어느 출력 링크로 보낼 것인가?

2. **Routing**
   - 출발지에서 목적지까지
   - 전체 경로를 어떻게 정할 것인가?

손필기식으로 말하면:

- forwarding = **갈림길 하나에서 어느 출구로 빠질지 정함**
- routing = **출발지부터 목적지까지 전체 길찾기**

## 1-2. Data plane vs Control plane

### Data plane
- 각 라우터 안에서 일어나는 로컬 처리
- 패킷이 들어오면
  - 헤더 확인
  - forwarding table 조회
  - 출력 포트 결정
- 매우 빠르게 동작해야 함
- 하드웨어 중심

### Control plane
- 네트워크 전체 관점의 논리
- 경로를 계산하고
- forwarding table을 만든다
- 비교적 느리지만 전체 정책을 담당
- 소프트웨어/알고리즘 중심

### 두 가지 control plane 방식

1. **Per-router control plane**
   - 각 라우터가 자체적으로 라우팅 알고리즘 수행

2. **SDN control plane**
   - 중앙/원격 controller가 계산
   - 라우터에는 결과 테이블을 설치

## 1-3. 서비스 모델: 인터넷은 무엇을 보장하는가?

네트워크 계층 서비스 모델은  
"송신자에서 수신자까지 가는 datagram channel이 무엇을 보장하느냐"를 묻는다.

예를 들어 이론상 가능한 보장:
- 반드시 도착
- 순서대로 도착
- 지연 40ms 이하 보장
- 최소 대역폭 보장

하지만 **인터넷 IP의 기본 모델은 best effort**다.

즉, 기본적으로는 다음을 보장하지 않는다.
- 도착 성공 보장 X
- 순서 보장 X
- 지연 보장 X
- 대역폭 보장 X

### 왜 이런 모델이 쓰였는가?

교안의 포인트:
- 구조가 단순해서 전 세계 확장에 유리
- 충분한 대역폭 공급, CDN, 분산 서비스 덕분에
  현실적으로는 "대부분의 경우 괜찮게" 동작
- 복잡한 보장은 응용 계층/전송 계층/분산 인프라가 보완

손필기식 표현:
- 인터넷은 기본적으로 **"최선을 다해 보내볼게"** 모델
- 대신 위 계층과 인프라가 이를 보완한다

## 1-4. ASCII 와이어프레임

```text
[Host] --(IP datagram)--> [Router] --(IP datagram)--> [Router] --> [Host]

네트워크 계층 질문
1) 이 라우터에서는 어디로 보낼까?  -> forwarding
2) 전체적으로 어느 길로 갈까?     -> routing
```

```text
Data plane   : 패킷 1개 들어왔을 때 즉시 처리
Control plane: 전체 라우팅 규칙/테이블 계산
```

## 1-5. 시험 포인트

- forwarding vs routing 차이 서술형 자주 나옴
- data plane vs control plane 비교형 자주 나옴
- 인터넷이 best effort라는 말의 의미:
  - "느리다"가 아니라
  - **보장을 하지 않는다**는 뜻
- best effort인데도 인터넷이 성공한 이유를
  "단순성 + 규모 확장 + 상위 계층/분산 인프라 보완"
  관점으로 설명할 수 있어야 함

---

# 2. 라우터 내부 구조와 포워딩 (p4-13 ~ p4-40)

## 2-1. 라우터의 큰 구조

라우터를 크게 보면 4부분이다.

1. **Input ports**
2. **Switching fabric**
3. **Output ports**
4. **Routing processor**

### 각 요소의 역할

- Input port
  - 비트 수신
  - 링크 계층 처리
  - forwarding table lookup
  - 필요 시 입력 큐잉

- Switching fabric
  - 입력 포트에서 출력 포트로 내부 전달

- Output port
  - 버퍼 저장
  - 스케줄링 후 링크로 전송

- Routing processor
  - control plane 담당
  - 라우팅/관리 기능 수행

교안 포인트:
- data plane은 매우 빠른 하드웨어 처리
- control plane은 비교적 느린 소프트웨어 처리

## 2-2. 입력 포트에서 하는 일

입력 포트에서는 패킷을 받아서
- 물리 계층 수신
- 링크 계층 처리
- 헤더를 보고 어느 출력 포트로 보낼지 lookup
- 스위치 패브릭이 바로 못 받으면 큐에 저장

여기서 목표는 **line speed**
- 들어오는 속도를 못 따라가면 병목이 생긴다

## 2-3. Destination-based forwarding vs Generalized forwarding

### Destination-based forwarding
- 전통적인 방식
- **목적지 IP 주소만 보고** 출력 포트 결정

### Generalized forwarding
- 더 넓은 방식
- 링크/네트워크/전송 계층의 여러 필드를 보고 판단 가능
- 예:
  - 목적지 IP
  - 출발지 IP
  - TCP/UDP 포트
  - VLAN
  - MAC 주소

손필기식 표현:
- 전통 방식 = **"목적지 주소만 보고 길 보냄"**
- 일반화 방식 = **"패킷의 여러 조건을 보고 규칙대로 행동"**

## 2-4. Longest Prefix Matching (p4-17 ~ p4-22)

이 부분은 시험에서 매우 중요하다.

포워딩 테이블에서 여러 엔트리가 동시에 맞을 수 있다.  
그럴 때 **가장 긴 prefix**가 맞는 엔트리를 선택한다.

### 왜 필요한가?

주소는 계층적으로 할당된다.
- 큰 네트워크 블록도 있고
- 그 안의 더 작은 하위 블록도 있다

그래서
- 큰 범위 규칙 하나
- 그 안의 예외/더 구체적 규칙 하나
가 동시에 존재할 수 있다.

이때 **더 구체적인 규칙이 우선**되어야 한다.

손필기식 표현:
- 넓게 맞는 주소보다
- **더 길게, 더 정확하게 맞는 주소가 이긴다**
- 즉, "더 구체적인 주소 규칙 우선"

### 핵심 감각

- prefix = 앞부분 비트가 얼마나 일치하느냐
- longest prefix = 가장 많이 일치한 엔트리 선택

## 2-5. Switching fabric 종류 (p4-23 ~ p4-28)

라우터 내부에서 패킷을 입력 → 출력으로 넘기는 방식은 3가지로 본다.

### 1) Memory 기반
- 초기 라우터 방식
- 입력 포트에서 메모리로 복사
- CPU/메모리 대역폭이 병목
- datagram 하나가 메모리를 두 번 건너는 느낌

### 2) Bus 기반
- 공유 버스를 통해 이동
- 구조는 단순
- 하지만 버스 대역폭 경쟁(bus contention) 발생

### 3) Interconnection network 기반
- crossbar, multistage switch 등
- 병렬성 활용 가능
- 고성능 라우터에 적합
- 큰 datagram을 작은 cell로 나누어 내부 스위칭 후 재조립 가능

손필기식 표현:
- memory: **CPU가 직접 챙기던 옛 방식**
- bus: **한 줄 통로를 다 같이 공유**
- interconnection: **여러 통로를 병렬로 구성**

## 2-6. Input port queuing과 HOL blocking (p4-29)

입력 포트 큐는
- 스위치 패브릭이 입력 속도를 못 따라갈 때 생긴다

대표 문제가 **HOL blocking**
- Head Of the Line blocking

뜻:
- 큐 맨 앞 패킷이 막혀 있으면
- 뒤 패킷도 못 나감
- 뒤 패킷은 다른 출력 포트로 갈 수 있어도 같이 멈춤

손필기식 표현:
- **맨 앞차가 좌회전 못 해서 막히면 뒤차도 다 못 감**

### 왜 중요할까?

- "뒤 패킷은 갈 수 있는데 못 가는" 비효율이 발생
- 입력 큐 구조만 잘못 잡아도 성능이 떨어진다

## 2-7. Output port queuing (p4-30 ~ p4-31)

출력 포트에서는
- 여러 입력 포트에서 온 패킷이 한 출력 링크로 몰릴 수 있다

만약 들어오는 속도 > 출력 링크 전송 속도 라면
- 버퍼 필요
- 대기열(queue) 발생
- 지연 증가
- 버퍼가 꽉 차면 손실 발생

즉, 출력 포트는
- buffering
- queueing delay
- packet drop
- scheduling
문제가 핵심이다.

## 2-8. Buffering과 Buffer Management (p4-32 ~ p4-33)

### 버퍼가 왜 필요한가?
순간적으로 패킷이 몰리면
출력 링크가 한 번에 다 못 보내므로
잠깐 저장할 공간이 필요하다.

### 너무 작은 버퍼
- 금방 overflow
- packet loss 증가

### 너무 큰 버퍼
- 손실은 줄 수 있어도
- 지연이 길어짐
- 실시간 앱, TCP 반응성에 악영향

즉,
- 버퍼는 무조건 크다고 좋은 게 아니다.

### Buffer management
버퍼가 찼을 때 무엇을 할지 정하는 규칙

대표 예:
- **tail drop**
  - 새로 들어온 패킷 버림
- **priority 기반 drop**
  - 우선순위 낮은 패킷 먼저 버림
- **marking**
  - ECN, RED처럼 혼잡 신호 표시 가능

## 2-9. Scheduling 정책 (p4-34 ~ p4-37)

이 부분은 "누가 먼저 전송 기회를 받는가" 문제다.

### 1) FCFS / FIFO
- 먼저 온 순서대로 보냄
- 가장 단순
- 공정해 보이지만
  긴 패킷/중요하지 않은 패킷이 앞에 있으면
  중요한 패킷도 기다려야 함

### 2) Priority scheduling
- 우선순위 높은 큐부터 먼저 전송
- 장점:
  - 중요한 트래픽 보호 가능
- 단점:
  - 낮은 우선순위는 굶을 수 있음(starvation)

### 3) Round Robin
- 클래스별로 한 바퀴씩 돌며 서비스
- 각 큐에 순서대로 기회 부여
- FCFS보다 클래스 간 균형감 있음

### 4) Weighted Fair Queuing (WFQ)
- RR의 일반화
- 각 클래스에 weight를 둔다
- weight가 큰 클래스가 더 많은 서비스 받음
- 최소 대역폭 보장 감각을 줄 수 있음

손필기식 표현:
- FCFS = 줄 선 순서
- Priority = VIP 먼저
- RR = 돌아가면서 한 번씩
- WFQ = 돌아가되, 중요도 따라 더 많이

## 2-10. Network Neutrality (p4-38 ~ p4-40)

교안은 여기서 기술이 사회 문제와 연결된다고 짚는다.

왜냐하면
- packet scheduling
- buffer management
같은 기술이

실제로는
- 특정 서비스 차별
- 속도 제한
- 유료 우선권
문제로 이어질 수 있기 때문이다.

즉,
라우터 스케줄링은 단순 구현 이슈가 아니라
**"누구에게 더 좋은 인터넷을 줄 것인가"**의 문제이기도 하다.

## 2-11. ASCII 와이어프레임

```text
[Input Port] --> [Switching Fabric] --> [Output Port]
       \                                 /
        \---- forwarding lookup --------/
                ^
                |
        [Routing Processor]
```

```text
HOL blocking

입력 큐:
[ A ][ B ][ C ]

A가 막힘 -> 뒤의 B, C도 못 나감
비록 B, C는 다른 출력 포트로 갈 수 있어도 대기
```

```text
스케줄링 비교

FCFS : 1 -> 2 -> 3 -> 4
PRI  : High 먼저, Low 나중
RR   : Q1 -> Q2 -> Q3 -> Q1 -> ...
WFQ  : Q1(2번) -> Q2(1번) -> Q3(1번) ...
```

## 2-12. 시험 포인트

- longest prefix matching은 거의 핵심
- HOL blocking 정의 서술 가능해야 함
- input queue와 output queue 차이 구분
- switching fabric 3종류 비교 가능해야 함
- scheduling 정책별 장단점 비교 가능해야 함
- network neutrality와 scheduling이 연결되는 이유를 말할 수 있으면 좋음

---

# 3. IP와 주소 체계의 핵심 (p4-41 ~ p4-62)

## 3-1. Internet에서 네트워크 계층이 하는 일

교안은 Internet 관점에서 네트워크 계층을 다시 보여준다.

핵심 구성:
- **IP**
  - datagram format
  - addressing
  - packet handling conventions
- **ICMP**
  - 에러 보고
  - 라우터 신호(signaling)

즉,
IP만 있다고 끝이 아니라  
ICMP 같은 보조 signaling도 있다.

## 3-2. IPv4 Datagram Format (p4-43)

IPv4 datagram 헤더에서 중요한 필드:

- version
- header length
- type of service (diffserv, ECN)
- total length
- identifier
- flags
- fragment offset
- TTL
- upper layer protocol
- header checksum
- source IP
- destination IP
- options

### 꼭 이해해야 할 필드

#### 1) TTL
- 라우터를 지날 때마다 1씩 감소
- 무한 루프 방지
- 0이 되면 폐기

손필기식 표현:
- **한 홉 지날 때마다 생명 1 깎임**

#### 2) Protocol
- 상위 계층이 무엇인지 표시
- 예: TCP, UDP

#### 3) Fragment 관련 필드
- datagram이 잘려서(fragmented) 갈 때
  - 어떤 조각인지
  - 어디에 놓일 조각인지
  추적하는 데 사용

#### 4) Header checksum
- 헤더 오류 검출용

### 오버헤드 감각
- TCP 20 bytes
- IP 20 bytes
- 합쳐서 40 bytes + 응용 계층 오버헤드

즉, 데이터만 보내는 게 아니라
헤더 비용도 계속 붙는다.

## 3-3. IP 주소는 누구에게 부여되는가? (p4-44 ~ p4-46)

중요:
- IP 주소는 **호스트 전체**가 아니라
- **인터페이스(interface)** 에 부여된다.

### interface란?
- 호스트/라우터와 물리 링크를 연결하는 지점

그래서
- 라우터는 인터페이스가 여러 개
- 호스트도 유선/무선 있으면 여러 인터페이스 가능

손필기식 표현:
- IP 주소는 "컴퓨터 주민번호"라기보다
- **연결 포트의 주소**에 가깝다

## 3-4. Subnet (p4-47 ~ p4-49)

### subnet이란?
중간 라우터를 거치지 않고
서로 직접 도달 가능한 인터페이스들의 집합

### 정의 레시피
교안 방식:
1. 각 인터페이스를 장치에서 떼어내어
2. 독립된 섬처럼 생각
3. 라우터를 거치지 않고 연결되는 한 덩어리가 subnet

### 주소 구조
IP 주소는 보통
- subnet part
- host part
로 나뉜다

예: /24
- 앞 24비트 = subnet part
- 뒤 8비트 = host part

손필기식 표현:
- 같은 subnet = **같은 동네**
- host part = **그 동네 안의 집 번호**

## 3-5. CIDR (p4-50)

CIDR = Classless InterDomain Routing

핵심:
- 주소의 네트워크 부분 길이를
- class A/B/C처럼 고정하지 않고
- **필요에 따라 유연하게 자른다**

표기:
- a.b.c.d/x

예:
- 200.23.16.0/23

뜻:
- 앞 23비트가 네트워크 부분
- 나머지가 호스트 부분

### 왜 중요할까?
- 주소를 더 효율적으로 배분 가능
- route aggregation 가능
- longest prefix matching과 자연스럽게 연결

## 3-6. 호스트는 IP를 어떻게 받는가? DHCP (p4-51 ~ p4-57)

호스트가 네트워크에 들어오면
보통 DHCP로 주소를 동적으로 받는다.

### DHCP의 목표
- 자동으로 IP 할당
- lease 갱신 가능
- 주소 재사용 가능
- 이동 사용자 지원

### 기본 4단계
1. **Discover**
2. **Offer**
3. **Request**
4. **ACK**

자주 DORA라고 외운다.

### 각 단계 의미

#### Discover
- "DHCP 서버 있나요?"
- 브로드캐스트로 찾음

#### Offer
- 서버가 "이 주소 써도 돼요" 제안

#### Request
- 클라이언트가 "그 주소 쓰겠습니다"

#### ACK
- 서버가 "좋아요, 그 주소 당신 것"

### DHCP가 주소 말고도 주는 것
- first-hop router 주소
- DNS 서버 주소
- subnet mask

즉,
DHCP는 단순 IP 배정기가 아니라
**초기 네트워크 설정 패키지**를 준다.

손필기식 표현:
- 컴퓨터가 랜선/와이파이에 붙으면
- DHCP가 **"너는 이 주소 쓰고, 게이트웨이는 얘고, DNS는 얘야"**
  라고 알려줌

## 3-7. 네트워크는 subnet prefix를 어떻게 받는가? (p4-58 ~ p4-62)

호스트가 host part를 받는 것과 별개로  
네트워크 자체도 subnet part를 받아야 한다.

흐름:
- ISP가 큰 주소 블록을 가짐
- 그 안에서 조직별로 더 작은 블록 분배
- 이런 계층적 할당이 route aggregation을 가능하게 함

### Route aggregation
여러 작은 네트워크를 하나의 큰 prefix로 묶어 광고

장점:
- 라우팅 테이블 간결화
- 광고 정보 감소

### More specific route
더 구체적인 경로가 있으면
그 경로가 우선될 수 있다.

이 역시 longest prefix matching과 연결된다.

### ICANN
- 전 세계 주소 자원 관리의 큰 축
- 지역 레지스트리 등을 통해 주소 블록 할당

### IPv4 주소 부족
- IPv4는 32비트라 한계
- 그래서 NAT가 버티게 해 주고
- 장기적으로는 IPv6가 필요

## 3-8. ASCII 와이어프레임

```text
IP 주소 구조
[ subnet part | host part ]

예: /24
[ 24비트 네트워크 | 8비트 호스트 ]
```

```text
DHCP (DORA)

Client                         DHCP Server
  |---- Discover ------------->|
  |<--- Offer -----------------|
  |---- Request -------------->|
  |<--- ACK -------------------|
```

```text
Route Aggregation

큰 블록: 200.23.16.0/20
 ├─ 200.23.16.0/23
 ├─ 200.23.18.0/23
 ├─ 200.23.20.0/23
 └─ ...
```

## 3-9. 시험 포인트

- IP 주소는 인터페이스에 부여된다는 점
- subnet 정의를 라우터 개입 여부로 설명 가능해야 함
- CIDR 표기 해석 가능해야 함
- DHCP 4단계 순서 정확히 쓰기
- route aggregation과 longest prefix matching 연결 설명
- IPv4 부족 → NAT/IPv6 필요성 연결

---

# 4. NAT: 주소 부족을 버티게 해 준 현실적 기술 (p4-63 ~ p4-68)

## 4-1. NAT의 기본 생각

NAT = Network Address Translation

핵심:
- 로컬 네트워크 내부 장치들은
  사설 IP 주소를 사용
- 외부 인터넷으로 나갈 때는
  **하나의 공인 IP 주소**를 공유

즉,
집 안에서는
- 10.0.0.x
- 192.168.x.x
같은 private IP 사용

밖으로 나갈 때는
- 공유기/NAT 장비가
  하나의 public IP로 바꿔서 내보낸다

## 4-2. 왜 필요한가?

### 장점
1. 공인 IP 하나로 여러 장치 사용 가능
2. 내부 주소 변경해도 외부에 알릴 필요 없음
3. ISP 바꿔도 내부 장치 주소 유지 쉬움
4. 외부에서 내부 장치가 직접 보이지 않아
   어느 정도 숨김 효과

손필기식 표현:
- NAT = **집 안 주소는 제각각, 밖에 나갈 땐 대표번호 하나 사용**

## 4-3. NAT 동작 원리

핵심은
- IP 주소만 바꾸는 것이 아니라
- **포트 번호까지 함께 바꾼다**

### 나갈 때
- (내부 source IP, source port)
  ->
- (공인 NAT IP, 새 port)

### 들어올 때
- 응답이
  (공인 NAT IP, 새 port) 로 돌아오면
- NAT table을 보고
- 원래 내부 IP:port로 되돌린다

즉,
NAT 장비는 translation table을 유지해야 한다.

## 4-4. 왜 port도 함께 바꾸는가?

공인 IP가 하나뿐이면
같은 시간에 여러 내부 장치가 외부와 통신할 때
구분이 안 된다.

그래서 port까지 재할당해서
- 누구의 연결인지 식별한다.

즉,
NAT는 사실상
- **address + port 변환 장치**

## 4-5. NAT의 논란

교안에서 NAT는 "현실적으로 많이 쓰지만 논란도 있는 기술"로 나온다.

논점:
- 라우터는 원래 3계층까지만 보는 게 자연스럽지 않나?
- 주소 부족 문제는 IPv6로 해결하는 게 정석 아닌가?
- NAT는 포트도 바꾸므로 종단간 원칙을 깨는 것 아닌가?
- NAT 뒤에 있는 서버에 외부가 먼저 접속하려면 어떻게 하나?
  -> NAT traversal 문제

즉,
NAT는 실용적이지만
인터넷의 원래 철학과는 약간 충돌한다.

## 4-6. ASCII 와이어프레임

```text
[10.0.0.1:3345] --\
[10.0.0.2:4000] --- [ NAT Router ] ---- Internet
[10.0.0.3:7777] --/

밖에서 볼 때:
모두 [138.76.29.7:서로 다른 포트] 로 보임
```

```text
NAT table 예시

공인측                내부측
138.76.29.7:5001  ->  10.0.0.1:3345
138.76.29.7:5002  ->  10.0.0.2:4000
```

## 4-7. 시험 포인트

- NAT가 왜 IPv4 부족 완화에 도움 되는지
- private IP / public IP 구분
- NAT가 IP와 port를 함께 바꾼다는 점
- NAT의 장점과 비판점 둘 다 정리
- NAT traversal 문제를 한 줄로 설명 가능하면 좋음

---

# 5. IPv6와 전환 방식 (p4-69 ~ p4-76)

## 5-1. IPv6가 왜 필요한가?

가장 직접적인 이유:
- **IPv4 32비트 주소 공간 부족**

추가 이유:
- 라우터 처리 속도 향상
- 헤더 단순화
- flow 단위 처리 지원 의도

## 5-2. IPv6의 핵심 특징

### 1) 주소 길이
- 128비트 주소 사용
- 주소 공간이 매우 큼

### 2) 헤더 단순화
IPv4와 비교해 빠진 것:

- 헤더 체크섬 없음
- 라우터 단편화/재조립 없음
- 기본 헤더 내 options 없음

### 왜 없앴는가?
라우터가 매 패킷마다 복잡한 계산을 덜 하게 해서
**포워딩 속도를 빠르게** 하려는 목적

손필기식 표현:
- IPv6는 주소만 커진 게 아니라
- **라우터가 빨리 처리하도록 헤더를 정리한 버전**

## 5-3. Hop Limit

IPv4의 TTL과 비슷한 역할
- 라우터를 지날 때마다 감소
- 무한 루프 방지

## 5-4. IPv4 -> IPv6 전환의 현실 문제

모든 라우터를 하루아침에 업그레이드할 수 없다.
교안 표현으로는 **no flag days**
- 어느 날 갑자기 전 세계가 한 번에 바뀌지 못함

그래서 혼합 환경이 생긴다.
- 어떤 구간은 IPv4
- 어떤 구간은 IPv6

## 5-5. Tunneling (p4-71 ~ p4-74)

핵심:
- IPv6 datagram을
- IPv4 datagram의 payload 안에 넣어서
- IPv4 구간을 통과시킨다

즉,
**packet within a packet**

### 흐름 감각
- IPv6 라우터 A에서 나옴
- 중간 IPv4 구간에서는
  IPv6 packet이 IPv4 껍데기 안에 들어감
- 반대쪽 IPv6 라우터에서 껍데기를 벗김
- 다시 순수 IPv6로 진행

손필기식 표현:
- IPv6 패킷이 IPv4 터널 안을 지나간다
- **택배 박스를 한 번 더 큰 박스에 넣어 보냄**

## 5-6. Adoption

교안은 IPv6 도입이 오래 걸렸다고 강조한다.
이 부분의 핵심은 수치보다 메시지다.

메시지:
- 기술적으로 필요하다고 바로 다 바뀌는 것은 아님
- 전 세계 인프라 교체는 매우 오래 걸린다

## 5-7. ASCII 와이어프레임

```text
IPv6 Host ---- IPv6 Router ==(IPv4 network)== IPv6 Router ---- IPv6 Host

중간 IPv4 구간:
[ IPv4 header | IPv6 datagram ]
```

```text
tunneling
원래 패킷 : [ IPv6 ]
터널 구간 : [ IPv4 [ IPv6 ] ]
터널 종료 : [ IPv6 ]
```

## 5-8. 시험 포인트

- IPv6 도입 이유 2~3개 말하기
- IPv6에서 빠진 필드의 의미
- 헤더 체크섬/라우터 단편화 제거 이유
- tunneling 정의 정확히 설명
- "packet within a packet" 감각 기억

---

# 6. Generalized Forwarding, SDN, OpenFlow (p4-77 ~ p4-87)

## 6-1. 왜 generalized forwarding이 필요한가?

전통 라우팅은
- 목적지 IP만 보고 보낸다

하지만 현대 네트워크는
- 보안
- 로드밸런싱
- 정책 기반 차단
- 트래픽 엔지니어링
등이 필요하다.

그래서
패킷의 여러 헤더 필드를 보고
더 다양한 동작을 하게 된다.

이것이 **match + action** 추상화다.

## 6-2. Match + Action의 뜻

### Match
- 패킷 헤더 필드가
- 어떤 패턴과 일치하는지 검사

### Action
일치하면 수행할 동작
- forward
- drop
- modify
- copy/log
- controller로 전송

즉,
- "이런 조건이면"
- "이렇게 행동해라"

## 6-3. Flow table의 구성

Flow table은 보통 다음 요소를 가진다.

- match
- action
- priority
- counters

### priority가 필요한 이유
패턴이 겹칠 수 있기 때문

예:
- src=10.1.*.*
- src=10.1.2.3

둘 다 맞으면
어느 규칙을 우선할지 정해야 한다.

### counters
- 패킷 수, 바이트 수 등 기록
- 모니터링/정책 판단에 활용

## 6-4. OpenFlow가 보는 필드

OpenFlow는 매우 다양한 헤더 필드를 match 대상으로 삼을 수 있다.

예:
- ingress port
- src/dst MAC
- VLAN
- IP src/dst
- IP protocol
- TCP/UDP src port
- TCP/UDP dst port

즉,
링크 계층 ~ 전송 계층까지 걸친다.

## 6-5. OpenFlow 예시 감각

### 예 1. 목적지 IP 기반 포워딩
- dest IP = 51.6.0.8 이면 port 6으로 보냄

### 예 2. 방화벽
- TCP dst port = 22 이면 drop
- 즉 ssh 차단

### 예 3. 특정 송신자 차단
- src IP = 128.119.1.1 이면 drop

즉,
OpenFlow는 router만 설명하는 도구가 아니라
switch, firewall, NAT까지
하나의 규칙 체계로 볼 수 있게 한다.

## 6-6. 왜 중요한가?

이 장의 포인트는
네트워크 장비를 단순한 "박스"로 보지 않고

- 입력 패킷의 특정 필드를 보고
- 규칙대로 처리하는
**프로그래밍 가능한 시스템**

으로 바라보게 만든다는 것이다.

교안 표현:
- simple form of network programmability

## 6-7. ASCII 와이어프레임

```text
Flow table

[ match ] ----------------> [ action ]
src=10.1.2.3                send to controller
dest=3.4.*.*                forward(2)
tcp dst port=22             drop
```

```text
Match + Action

패킷 도착
   |
헤더 검사
   |
조건 일치?
   |
   +--> forward
   +--> drop
   +--> modify
   +--> controller 전송
```

## 6-8. 시험 포인트

- generalized forwarding 정의
- destination-based forwarding과 차이
- flow table 요소 4개
- priority가 왜 필요한지
- OpenFlow가 여러 계층 헤더를 볼 수 있다는 점
- router / switch / firewall / NAT를 match+action 관점으로 묶어 설명 가능해야 함

---

# 7. Middleboxes와 인터넷 구조 철학 (p4-88 ~ p4-99)

## 7-1. Middlebox란?

정의:
- source와 destination 사이 경로에 있으면서
- 표준 IP router 기능 외의 추가 기능을 하는 중간 장치

대표 예:
- NAT
- firewall
- IDS
- load balancer
- cache
- application-specific box

즉,
단순 전달만 하는 것이 아니라
패킷을 검사/수정/차단/분산/캐싱하는 장치들

## 7-2. 왜 middlebox가 많아졌는가?

실제 네트워크는 단순 전달만으로 부족하기 때문이다.

필요한 것:
- 보안
- 주소 변환
- 로드 분산
- 성능 향상
- 캐싱
- 정책 제어

그래서 인터넷 한가운데에도
복잡성이 많이 들어오게 되었다.

## 7-3. 최근 흐름

교안 포인트:
- 전용 proprietary box에서
- whitebox + open API + software 중심으로 이동
- SDN
- NFV(Network Functions Virtualization)
와 연결

즉,
기능은 더 많아지지만
그 구현은 하드웨어 고정형보다
소프트웨어/가상화 중심으로 이동 중

## 7-4. IP Hourglass

이 그림은 인터넷 구조 철학의 핵심이다.

위:
- HTTP, SMTP, QUIC, DASH 등 많은 상위 프로토콜

가운데:
- **IP**
- 얇은 허리(thin waist)

아래:
- Ethernet, WiFi, Bluetooth, 광/무선/구리 등 다양한 링크/물리 기술

핵심 의미:
- 위아래는 다양해도
- 가운데 IP 하나로 연결됨

손필기식 표현:
- 인터넷은 가운데 IP 하나가 허리처럼 잡고 있어서
  위아래 기술이 다양해도 전체가 이어진다

## 7-5. Middle age의 "love handles"

교안은 유머 있게
원래 얇은 허리였던 인터넷에
middlebox가 늘며 옆구리가 붙었다고 표현한다.

뜻:
- 이상적인 단순 IP 구조 위에
- 실제로는 중간 기능 장치가 많이 끼어들었다

## 7-6. End-to-End Argument

이건 매우 중요한 철학이다.

핵심 주장:
어떤 기능은
- 네트워크 안쪽에서 완전히 해결할 수 없고
- **끝단(application/end host)의 지식과 도움까지 있어야**
  제대로 구현 가능하다.

예:
- reliable data transfer
- congestion control

즉,
가능하면 복잡성/지능은 네트워크 내부보다
**종단(edge)** 에 두는 것이 낫다.

### 왜인가?
네트워크 내부는
- 일반적인 공통 기능만 알 수 있음
- 응용의 구체적 요구를 모름

반면 종단은
- 어떤 데이터인지
- 어느 정도 신뢰성/지연이 필요한지
- 최종 성공 조건이 무엇인지
를 안다.

손필기식 표현:
- **정답을 가장 잘 아는 쪽은 끝단**
- 그래서 중요한 지능은 edge에 두자는 철학

## 7-7. Where’s the intelligence?

교안 흐름:
- 옛 전화망: 지능이 네트워크 중심
- 초기 인터넷: 지능이 edge 중심
- 현대 인터넷: edge 인프라 + programmable middlebox가 함께 존재

즉,
완전히 한쪽만이 아니라
현대 인터넷은 다시 복잡해지고 있다.

## 7-8. ASCII 와이어프레임

```text
IP Hourglass

   HTTP  SMTP  QUIC  DASH
          \   |   /
             IP
       /   /  |  \   \
 Ethernet WiFi LTE Bluetooth ...
```

```text
End-to-End Argument

[Host] --- [Router] --- [Router] --- [Host]

어떤 기능은 중간 라우터만으로 완전 구현 불가
끝단이 데이터 의미와 성공 조건을 가장 잘 안다
```

```text
Middleboxes 예시

Client --> Firewall --> NAT --> Load Balancer --> Cache --> Server
```

## 7-9. 시험 포인트

- middlebox 정의 정확히 쓰기
- 대표 middlebox 예시 3~4개
- IP hourglass 의미 설명
- thin waist가 왜 중요한지
- end-to-end argument의 핵심 문장 설명
- NAT가 왜 end-to-end 철학과 충돌하는지도 연결 가능

---

# 8. 보충: IP fragmentation / reassembly (추가 슬라이드 p4-100)

## 8-1. 왜 단편화가 필요한가?

링크마다 MTU(Maximum Transfer Unit)가 다르다.
- 한 링크에서 보낼 수 있는 최대 프레임 크기가 다름

그래서 어떤 링크를 지나려면
큰 IP datagram을 잘라야 할 수 있다.

## 8-2. fragmentation의 의미

- 큰 IP datagram을 여러 조각으로 나눔
- 각 조각도 datagram처럼 전달
- 목적지에서 다시 재조립

교안 포인트:
- **reassembled only at destination**
- 중간 라우터마다 다시 조립하는 것이 아님

## 8-3. 관련 필드
IPv4 헤더의
- identifier
- flags
- fragment offset
이 조각들을 구분하고 순서를 맞추는 데 중요

손필기식 표현:
- 큰 택배를 여러 상자로 나눠 보내고
- 최종 목적지에서만 다시 합친다

## 8-4. 시험 포인트

- MTU 뜻
- fragmentation 이유
- 재조립은 목적지에서만 한다는 점
- IPv6는 라우터 단편화를 기본 헤더 수준에서 제거했다는 점과 비교 가능

---

# 마지막. 헷갈리는 비교 정리

## 1. Forwarding vs Routing

| 구분 | Forwarding | Routing |
|---|---|---|
| 의미 | 한 라우터 안에서 입력 → 출력 결정 | 전체 경로 계산 |
| 범위 | 로컬 | 네트워크 전체 |
| 속도 | 매우 빠름 | 상대적으로 느림 |
| 비유 | 갈림길에서 어느 출구로 나갈지 | 전체 여행 경로 짜기 |

## 2. Data plane vs Control plane

| 구분 | Data plane | Control plane |
|---|---|---|
| 역할 | 패킷 실제 처리 | 경로/규칙 계산 |
| 위치 감각 | 각 라우터 내부 | 네트워크 전체 논리 |
| 구현 감각 | 하드웨어 중심 | 소프트웨어/알고리즘 중심 |

## 3. Destination-based vs Generalized forwarding

| 구분 | Destination-based | Generalized |
|---|---|---|
| 기준 | 목적지 IP 중심 | 여러 계층 헤더 필드 |
| 동작 | 주로 forward | forward/drop/modify/controller |
| 활용 | 전통 라우터 | SDN/OpenFlow, firewall, NAT 등 |

## 4. Input queue vs Output queue

| 구분 | Input queue | Output queue |
|---|---|---|
| 생기는 이유 | fabric이 입력 속도를 못 따라감 | 출력 링크가 몰림 |
| 대표 문제 | HOL blocking | buffering, loss, scheduling |
| 핵심 질문 | 앞에서 막혀 뒤도 못 가는가 | 누구를 먼저 보낼 것인가 |

## 5. FCFS vs Priority vs RR vs WFQ

| 정책 | 핵심 |
|---|---|
| FCFS | 먼저 온 순서대로 |
| Priority | 중요한 것 먼저 |
| RR | 큐별로 돌아가며 |
| WFQ | 돌아가되 가중치 반영 |

## 6. Subnet vs Host part

| 구분 | 의미 |
|---|---|
| subnet part | 같은 네트워크/동네를 나타냄 |
| host part | 그 subnet 안의 개별 장치 |

## 7. Private IP vs Public IP

| 구분 | Private IP | Public IP |
|---|---|---|
| 범위 | 내부 네트워크 전용 | 인터넷 전역 식별 가능 |
| 예 | 10/8, 172.16/12, 192.168/16 | ISP가 할당한 공인 주소 |
| NAT와 관계 | 내부 장치가 사용 | 외부에 대표로 보이는 주소 |

## 8. IPv4 vs IPv6

| 구분 | IPv4 | IPv6 |
|---|---|---|
| 주소 길이 | 32비트 | 128비트 |
| 체크섬 | 있음 | 없음 |
| 라우터 단편화 | 있음 | 기본 헤더 수준에선 없음 |
| 주소 부족 | 심각 | 크게 완화 |
| 전환 | 기존 체계 | tunneling 등 필요 |

## 9. NAT의 장점 vs 단점

| 장점 | 단점 |
|---|---|
| 공인 IP 절약 | end-to-end 원칙 약화 |
| 내부 주소 변경 쉬움 | NAT traversal 문제 |
| 내부 장치 숨김 효과 | 포트 조작 등 구조 복잡성 |

## 10. End-to-End vs In-network

| 구분 | End-to-End | In-network |
|---|---|---|
| 지능 위치 | 끝단 | 네트워크 내부 |
| 장점 | 응용 요구 반영 쉬움 | 공통 기능 가속 가능 |
| 철학 | 인터넷의 기본 철학에 가까움 | middlebox 확장으로 점점 증가 |

---

# 마지막 체크리스트

- [ ] forwarding / routing 차이를 말할 수 있다
- [ ] data plane / control plane 차이를 말할 수 있다
- [ ] 인터넷 best effort의 의미를 안다
- [ ] longest prefix matching을 설명할 수 있다
- [ ] HOL blocking을 설명할 수 있다
- [ ] switching fabric 3종류를 구분할 수 있다
- [ ] FCFS / Priority / RR / WFQ를 비교할 수 있다
- [ ] IP 주소는 인터페이스에 부여된다는 점을 안다
- [ ] subnet, CIDR, /24 같은 표기를 해석할 수 있다
- [ ] DHCP의 DORA 4단계를 외웠다
- [ ] NAT가 왜 필요한지와 어떻게 동작하는지 안다
- [ ] IPv6의 핵심 차이를 말할 수 있다
- [ ] tunneling을 packet within a packet으로 설명할 수 있다
- [ ] match + action의 뜻을 설명할 수 있다
- [ ] OpenFlow가 여러 계층 헤더를 본다는 점을 안다
- [ ] middlebox 예시를 3개 이상 말할 수 있다
- [ ] IP hourglass와 end-to-end argument를 설명할 수 있다
- [ ] fragmentation이 왜 생기고 어디서 재조립되는지 안다

---

## 한 줄 총정리

Chapter 4는  
**패킷을 어디로 어떻게 보낼지**를  
라우터 내부 구조, 주소 체계, NAT, IPv6, SDN, middlebox까지 연결해서 보는 장이다.

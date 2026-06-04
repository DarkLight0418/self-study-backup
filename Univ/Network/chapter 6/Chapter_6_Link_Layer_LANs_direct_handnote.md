# Chapter 6 Link Layer and LANs 손필기 전사용 노트

> 기준 자료: `Chapter_6_v80_260526_091707.pdf`  
> 작성 방식: 교안 순서를 그대로 베끼지 않고, 개념 단위로 묶어 바로 필기 가능한 형태로 재구성

---

## 0. 먼저 외울 핵심 문장

- Link Layer는 **한 노드에서 바로 옆 노드까지** 데이터그램을 프레임으로 옮기는 계층이다.
- 노드 = host + router, 링크 = 인접 노드를 연결하는 통신 채널이다.
- Network Layer는 목적지까지의 전체 경로를 보지만, Link Layer는 **현재 링크 하나**만 본다.
- Link Layer의 PDU는 **frame**이고, frame 안에 IP datagram이 들어간다.
- MAC 주소는 같은 LAN 안에서 프레임을 보내기 위한 **로컬 주소**이다.
- IP 주소는 계층 3 주소, MAC 주소는 계층 2 주소이다.
- ARP는 **IP 주소를 알고 있을 때 MAC 주소를 찾는 프로토콜**이다.
- Error detection은 오류를 “줄이는” 것이지 100% 완벽히 보장하는 것은 아니다.
- CRC는 Ethernet/WiFi에서 많이 쓰이는 강력한 오류 검출 방식이다.
- Multiple access는 여러 노드가 하나의 공유 채널을 어떻게 나눠 쓸지 정하는 문제이다.
- MAC 프로토콜은 크게 `채널 분할 / 랜덤 접근 / 순서 기반`으로 나뉜다.
- ALOHA는 단순하지만 충돌이 많고, CSMA는 먼저 듣고 보내므로 더 효율적이다.
- CSMA/CD는 충돌을 감지하면 즉시 중단하고 backoff 후 재전송한다.
- Ethernet은 대표적인 유선 LAN 기술이며, 현재는 switch 기반 구조가 일반적이다.
- Switch는 MAC 주소를 학습하여 필요한 포트로만 frame을 전달한다.
- VLAN은 하나의 물리 LAN을 여러 개의 논리 LAN처럼 나누는 기술이다.
- MPLS는 IP 주소 대신 짧은 label을 보고 빠르게 전달하는 방식이다.
- Data center network는 대규모 서버, 다중 경로, 부하 분산, 신뢰성이 핵심이다.
- 웹 요청 하나에도 DHCP, ARP, DNS, TCP, HTTP, Ethernet, IP가 계층별로 함께 동작한다.

---

## 1. Link Layer 큰 그림 (p4 ~ p9)

- Link Layer는 **인접한 두 노드 사이**에서 IP datagram을 전달한다.
- 여기서 노드는 host와 router를 모두 포함한다.
- 링크는 wired link, wireless link, LAN처럼 인접 장치를 연결하는 통신 구간이다.
- Link Layer packet은 **frame**이라고 부른다.

```text
[Application]
[Transport ]  segment
[Network   ]  datagram
[Link      ]  frame
[Physical  ]  bits
```

- 같은 IP datagram이라도 경로 중 각 링크마다 다른 link protocol을 사용할 수 있다.
  - 첫 링크: WiFi
  - 다음 링크: Ethernet
  - 또 다른 링크: 광케이블 기반 링크
- 손필기식으로는 이렇게 기억하면 좋다.
  - **IP datagram = 여행객**
  - **각 링크 = 이동 수단 하나**
  - **Link Layer = 이번 구간을 태워 보내는 역할**

```text
[Host A] --WiFi frame--> [Router]
[Router] --Ethernet frame--> [Router]
[Router] --다른 link frame--> [Host B]

IP datagram은 계속 이동하지만,
각 링크마다 frame 포장은 새로 바뀐다.
```

- Link Layer 주요 서비스
  - Framing: datagram을 frame에 넣음
  - Link access: 공유 매체에서 언제 보낼지 결정
  - MAC addressing: 같은 LAN 안에서 목적지 interface 식별
  - Error detection/correction: 비트 오류 탐지 또는 수정
  - Flow control: 인접 노드끼리 속도 조절
  - Half-duplex / Full-duplex: 동시에 양방향 전송 가능 여부

- 구현 위치
  - 주로 NIC(Network Interface Card)에 구현된다.
  - Ethernet 카드, WiFi 칩 등이 Link + Physical 계층을 담당한다.
  - CPU만의 일이 아니라 하드웨어, 소프트웨어, 펌웨어가 함께 처리한다.

```text
송신 측 NIC
IP datagram 받음
   ↓
frame으로 캡슐화
   ↓
오류 검출 비트 추가
   ↓
물리 링크로 전송

수신 측 NIC
frame 수신
   ↓
오류 확인
   ↓
IP datagram 추출
   ↓
Network Layer로 전달
```

- 시험 포인트
  - Link Layer는 end-to-end 전체가 아니라 **hop-to-hop / adjacent node** 중심이다.
  - IP 주소와 MAC 주소의 목적을 구분해야 한다.
  - frame은 링크마다 새로 만들어질 수 있다.

---

## 2. Error Detection and Correction (p11 ~ p15)

- 링크를 지나면 신호 감쇠, 잡음, 간섭 때문에 bit error가 생길 수 있다.
- 그래서 frame에는 데이터 `D`와 함께 오류 검출/수정 비트 `EDC`가 붙는다.

```text
송신 frame: [ D | EDC ]
                 ↓ noisy link
수신 frame: [ D'| EDC' ]
                 ↓
        오류가 있는지 검사
```

- EDC는 오류를 찾기 위한 redundancy, 즉 여분의 확인 정보이다.
- 단, 오류 검출은 100% 완벽하지 않다.
  - EDC가 길고 정교할수록 검출 능력은 좋아진다.
  - 그래도 특정 오류 패턴은 놓칠 수 있다.

### Parity Checking

- Single bit parity
  - 데이터 안의 1의 개수가 짝수/홀수가 되도록 parity bit를 붙인다.
  - 단일 bit 오류 검출에는 간단하고 유용하다.
  - 하지만 2개 bit가 동시에 바뀌면 놓칠 수 있다.

```text
데이터: 0111000110101011
parity bit 추가 → 0111000110101011 | 1

목표: 1의 개수를 짝수 또는 홀수 규칙에 맞춤
```

- Two-dimensional parity
  - 행과 열 방향으로 parity를 붙인다.
  - 단일 bit 오류는 위치까지 찾아 수정할 수 있다.

```text
원본 데이터 비트들을 표처럼 배치

      열 parity
        ↓
[1 0 1 1 | p]
[0 1 1 0 | p]
[1 1 0 0 | p]
--------------
[p p p p]
행 parity

행과 열이 동시에 어긋나는 교차점 = 오류 위치
```

### Internet Checksum

- 데이터를 16-bit 단위 정수처럼 보고 더한 뒤 checksum을 만든다.
- 수신자는 같은 방식으로 계산해서 checksum과 비교한다.
- Transport Layer의 UDP checksum에서 봤던 방식이다.
- 장점은 단순함, 단점은 보호 능력이 CRC보다 약하다는 점이다.

### CRC, Cyclic Redundancy Check

- CRC는 Link Layer에서 특히 중요한 강력한 오류 검출 방식이다.
- Ethernet과 802.11 WiFi에서 널리 사용된다.
- 핵심 생각:
  - 데이터 `D` 뒤에 `r`개의 CRC bit `R`을 붙인다.
  - 송신자는 `<D, R>`이 generator `G`로 나누어떨어지게 `R`을 선택한다.
  - 수신자는 같은 `G`로 나눠 보고 나머지가 0이 아니면 오류로 판단한다.

```text
D = data bits
G = generator, r+1 bits
R = r CRC bits

목표:
<D, R> 이 G로 정확히 나누어떨어지게 만들기
```

- 수식 감각
  - `D · 2^r XOR R = nG`
  - `D · 2^r`은 데이터 뒤에 r개의 0을 붙인 값이다.
  - `R`은 나눗셈의 나머지를 이용해 만든 CRC bit이다.
  - 수신자는 `<D, R> ÷ G`의 나머지를 확인한다.

- 시험 포인트
  - Parity: 단순, 약함
  - Checksum: 구현 쉬움, 중간 정도
  - CRC: 실무에서 강력하게 사용, burst error 검출에 강함

---

## 3. Multiple Access Protocols 큰 분류 (p17 ~ p20)

- Multiple access는 **여러 노드가 하나의 공유 채널을 동시에 쓰려 할 때** 생기는 문제이다.
- 동시에 두 노드 이상이 전송하면 신호가 겹쳐 collision이 생긴다.
- 그래서 MAC, Multiple Access Control protocol이 필요하다.

```text
공유 채널

Node A ----\
Node B -----+----> 하나의 broadcast medium
Node C ----/

동시에 보내면 collision 발생
```

- 이상적인 MAC 프로토콜 조건
  - 한 노드만 보내면 전체 속도 `R` 사용
  - M개 노드가 보내면 평균 `R/M`씩 공정하게 사용
  - 중앙 관리자 없이 분산적으로 동작
  - 동기화 부담이 작고 단순함

- MAC 프로토콜 3분류

| 분류 | 핵심 생각 | 장점 | 약점 |
|---|---|---|---|
| Channel partitioning | 시간/주파수/코드로 미리 나눔 | 고부하에서 공정 | 사용자가 적으면 낭비 |
| Random access | 일단 보내고 충돌 시 복구 | 저부하에서 효율 | 고부하에서 충돌 증가 |
| Taking turns | 차례대로 보냄 | 공정성과 효율 절충 | 제어 오버헤드 |

- 손필기식 표현
  - Channel partitioning = “자리표를 미리 나눠줌”
  - Random access = “말하고 싶으면 말하되, 겹치면 다시 시도”
  - Taking turns = “순번표 돌리기”

---

## 4. Channel Partitioning: TDMA / FDMA (p21 ~ p22)

- Channel partitioning은 공유 채널을 작은 조각으로 나누어 각 노드에 배정한다.

### TDMA

- TDMA = Time Division Multiple Access
- 시간을 slot으로 나누고, 각 station이 정해진 시간 slot에만 보낸다.

```text
시간 방향 →
| 1 | 2 | 3 | 4 | 5 | 6 | 1 | 2 | 3 | 4 | 5 | 6 |
  ↑       ↑   ↑           ↑       ↑   ↑
  전송    전송 전송        전송    전송 전송

보낼 데이터가 없는 slot은 idle
```

- 장점: 충돌이 거의 없고 공정하다.
- 단점: 특정 노드가 보낼 데이터가 없어도 해당 slot은 낭비된다.

### FDMA

- FDMA = Frequency Division Multiple Access
- 주파수 대역을 나누고, 각 station이 자기 주파수 band를 사용한다.

```text
주파수 ↑
Band 6 | idle
Band 5 | idle
Band 4 | node 4
Band 3 | node 3
Band 2 | idle
Band 1 | node 1
        시간 →
```

- 장점: 동시에 전송 가능하고 충돌이 적다.
- 단점: 사용하지 않는 주파수 band가 생기면 낭비된다.

- 시험 포인트
  - TDMA는 시간 분할, FDMA는 주파수 분할이다.
  - 둘 다 “미리 나눠주는 방식”이라 고부하에는 안정적이지만 저부하에는 비효율적이다.

---

## 5. Random Access: ALOHA, CSMA, CSMA/CD (p23 ~ p32, p105)

- Random access는 채널을 미리 나누지 않는다.
- 노드는 보낼 frame이 생기면 전송을 시도한다.
- 충돌이 생기면 일정 규칙에 따라 재시도한다.

### Slotted ALOHA

- 시간을 slot으로 나누고, 노드는 slot 시작점에서만 전송한다.
- 충돌이 없으면 성공, 충돌이 있으면 확률 `p`로 다음 slot에서 재전송한다.

```text
slot: | 1 | 2 | 3 | 4 | 5 |
A:       send       retry
B:       send
결과:    충돌        성공 가능
```

- 성공 확률 감각
  - N개 노드가 있고 각 노드가 확률 p로 전송할 때,
  - 특정 노드 성공 확률: `p(1-p)^(N-1)`
  - 어떤 노드든 성공할 확률: `Np(1-p)^(N-1)`
  - 노드가 많을 때 최대 효율은 약 `1/e ≈ 0.37`

- 의미
  - 최대로 잘 맞춰도 유용한 전송에 쓰이는 slot은 약 37% 수준이다.
  - 나머지는 충돌 또는 빈 slot로 낭비된다.

### Pure ALOHA

- slot 동기화 없이 frame이 생기면 즉시 보낸다.
- 단순하지만 충돌 가능 구간이 더 넓다.
- 최대 효율은 약 `1/(2e) ≈ 0.18`이다.

```text
t0에 보낸 frame은
[t0 - 1, t0 + 1] 근처에서 시작한 다른 frame과 겹칠 수 있음
→ 충돌 구간이 slotted ALOHA보다 큼
```

### CSMA

- CSMA = Carrier Sense Multiple Access
- 보내기 전에 먼저 채널을 듣는다.
  - idle이면 전송
  - busy이면 기다림
- 손필기식 표현: **“말하기 전에 남이 말하는지 먼저 듣기”**

```text
채널 감지
   ↓
idle? ── yes → 전송
  │
  no
  ↓
기다림
```

- 하지만 충돌이 완전히 사라지는 것은 아니다.
- 이유: propagation delay 때문이다.
  - A가 막 전송을 시작했는데, 멀리 있는 B는 아직 그 신호를 듣지 못할 수 있다.
  - B도 동시에 전송하면 collision 발생.

### CSMA/CD

- CSMA/CD = CSMA with Collision Detection
- Ethernet에서 쓰인 대표 방식이다.
- 충돌을 감지하면 frame 전송을 끝까지 하지 않고 즉시 중단한다.
- 그 뒤 jam signal을 보내고 binary exponential backoff 후 재시도한다.

```text
1. NIC가 datagram을 받아 Ethernet frame 생성
2. 채널 감지
   - idle이면 전송
   - busy이면 기다렸다가 전송
3. 충돌 없이 끝까지 보내면 완료
4. 보내는 중 충돌 감지 → 중단 + jam signal
5. backoff 후 다시 2번으로
```

- Binary exponential backoff
  - m번째 충돌 후 `K ∈ {0, 1, 2, ..., 2^m - 1}` 중 랜덤 선택
  - `K × 512 bit times`만큼 기다린 뒤 재시도
  - 충돌이 많을수록 기다릴 수 있는 범위가 커진다.

- CSMA/CD 효율 식
  - `Tprop` = LAN에서 두 노드 사이 최대 전파 지연
  - `Ttrans` = 최대 frame 전송 시간
  - 효율은 대략 `1 / (1 + 5Tprop/Ttrans)` 형태로 이해한다.
  - `Tprop`이 작을수록, `Ttrans`가 클수록 효율은 좋아진다.

- 시험 포인트
  - ALOHA: 단순, 충돌 많음
  - CSMA: 먼저 듣고 보냄
  - CSMA/CD: 충돌 감지 후 중단
  - 유선은 collision detection이 비교적 쉽고, 무선은 어렵다.

---

## 6. Taking Turns와 Cable Access (p33 ~ p38)

- Taking turns 방식은 channel partitioning과 random access의 중간 성격이다.
- 목표는 고부하에서는 공정하게, 저부하에서는 낭비를 줄이는 것이다.

### Polling

- master node가 각 slave에게 차례대로 “보낼 차례”를 준다.

```text
[Master]
   ├─ poll → [Node 1] → data
   ├─ poll → [Node 2] → data 없음
   └─ poll → [Node 3] → data
```

- 장점: 충돌을 줄이고 순서를 통제하기 쉽다.
- 단점: polling overhead, 지연, master 장애 시 전체 문제.

### Token Passing

- token이라는 제어 메시지를 가진 노드만 전송할 수 있다.

```text
[Node A] --token--> [Node B] --token--> [Node C]
   ↑                                      ↓
   └--------------- token ---------------┘
```

- 장점: 공정한 순서 제어.
- 단점: token overhead, token 분실/장애 문제.

### Cable Access Network / DOCSIS

- Cable access network는 FDM, TDM, random access가 섞여 있다.
- Downstream은 여러 주파수 채널로 broadcast된다.
- Upstream은 일부 slot이 할당되고, 일부는 contention/random access로 요청한다.
- DOCSIS는 cable modem에서 쓰이는 데이터 전송 규격이다.

- 시험 포인트
  - Polling은 master 중심.
  - Token passing은 token 중심.
  - DOCSIS는 FDM + TDM + random access 혼합 구조로 이해한다.

---

## 7. MAC Address와 ARP (p40 ~ p46)

- IP 주소는 Network Layer 주소이고, 목적지 네트워크까지 찾아가는 데 사용된다.
- MAC 주소는 Link Layer 주소이고, 같은 LAN 안에서 interface끼리 frame을 전달하는 데 사용된다.

| 구분 | IP 주소 | MAC 주소 |
|---|---|---|
| 계층 | Network Layer, L3 | Link Layer, L2 |
| 길이 | IPv4 기준 32-bit | 보통 48-bit |
| 역할 | 네트워크 간 라우팅 | 같은 LAN 내부 전달 |
| 성격 | 위치 기반, subnet에 의존 | flat address, NIC 중심 |
| 비유 | 집 주소 | 주민등록번호/장치 고유번호 |

- MAC 주소는 보통 16진수로 표현한다.

```text
예: 1A-2F-BB-76-09-AD
각 16진수 한 자리 = 4 bit
총 12자리 × 4 bit = 48 bit
```

### ARP

- ARP = Address Resolution Protocol
- 질문: “IP 주소는 아는데, 같은 LAN에서 frame을 보내려면 목적지 MAC 주소를 어떻게 알까?”
- 답: ARP를 사용한다.

- 각 host/router는 ARP table을 가진다.

```text
ARP table
IP address        MAC address              TTL
137.196.7.23      71-65-F7-2B-08-53        20분 정도
```

- TTL은 해당 mapping을 얼마나 오래 기억할지 나타낸다.

### ARP 동작 흐름

```text
A가 B에게 보내고 싶음
A는 B의 IP는 알고 있지만 B의 MAC을 모름

1. A → LAN 전체 broadcast
   "이 IP 주소 가진 장치 누구야? MAC 알려줘"
   destination MAC = FF-FF-FF-FF-FF-FF

2. B만 자신의 IP임을 확인

3. B → A에게 unicast ARP reply
   "내 MAC 주소는 이것"

4. A는 ARP table에 저장 후 frame 전송
```

- 손필기식 표현
  - **IP는 목적지 건물 주소**
  - **MAC은 같은 건물 안에서 실제 받을 사람/장치 번호**
  - **ARP는 IP를 MAC으로 바꾸는 로컬 조회 과정**

- 시험 포인트
  - ARP request는 broadcast.
  - ARP reply는 보통 unicast.
  - ARP는 같은 LAN 안에서 의미가 크다.
  - 다른 subnet으로 갈 때는 최종 목적지 MAC이 아니라 **first-hop router의 MAC**이 필요하다.

---

## 8. 다른 Subnet으로 보낼 때 주소 변화 (p47 ~ p52)

- A가 다른 subnet의 B에게 IP datagram을 보낼 때, A는 B에게 직접 Ethernet frame을 보낼 수 없다.
- A는 먼저 default gateway/router R에게 frame을 보낸다.

```text
A ---- LAN1 ---- R ---- LAN2 ---- B

A의 목표: B에게 IP datagram 전달
하지만 첫 링크에서 frame 목적지 MAC은 B가 아니라 R의 MAC
```

- 핵심은 IP 주소와 MAC 주소의 변화 범위이다.

| 항목 | A → R 구간 | R → B 구간 |
|---|---|---|
| IP source | A IP | A IP |
| IP destination | B IP | B IP |
| Ethernet source MAC | A MAC | R의 LAN2 MAC |
| Ethernet destination MAC | R의 LAN1 MAC | B MAC |

- IP datagram의 source/destination IP는 end-to-end로 유지된다.
- 하지만 Ethernet frame의 source/destination MAC은 링크마다 바뀐다.

```text
A가 만든 IP datagram
[IP src=A, IP dst=B]

A → R frame
[MAC src=A, MAC dst=R] [IP src=A, IP dst=B]

R → B frame
[MAC src=R, MAC dst=B] [IP src=A, IP dst=B]
```

- 시험 포인트
  - 다른 subnet으로 갈 때 ARP로 찾는 MAC은 최종 목적지 B가 아니라 다음 홉 R이다.
  - Router를 지날 때 frame은 새로 캡슐화된다.
  - IP 주소는 목적지까지 유지, MAC 주소는 hop마다 변경.

---

## 9. Ethernet 기본 구조 (p54 ~ p59)

- Ethernet은 대표적인 유선 LAN 기술이다.
- 초기에는 bus topology가 흔했지만, 현재는 switch topology가 일반적이다.

```text
과거 bus Ethernet
A ---- B ---- C ---- D
모두 같은 collision domain

현재 switched Ethernet
        [Switch]
       /   |   \
      A    B    C
각 링크가 분리되어 충돌 가능성 감소
```

### Ethernet Frame Structure

```text
+----------+----------+----------+------+-----+
| Preamble | Dest MAC | Src MAC  | Type |Data | CRC |
+----------+----------+----------+------+-----+
```

- Preamble
  - 수신자와 송신자의 clock 동기화를 돕는 시작 패턴.
- Destination MAC
  - frame을 받을 interface의 MAC 주소.
- Source MAC
  - frame을 보낸 interface의 MAC 주소.
- Type
  - payload에 들어 있는 상위 계층 프로토콜 식별.
  - 예: IP datagram.
- Data
  - 실제 payload. 보통 IP datagram이 들어감.
- CRC
  - 오류 검출용.

### Ethernet의 성격

- Connectionless
  - 송신 NIC와 수신 NIC 사이에 handshake를 하지 않는다.
- Unreliable
  - 수신 NIC가 ACK/NAK를 보내지 않는다.
  - 손실 frame 복구는 TCP 같은 상위 계층이 맡거나, 아니면 손실된 채로 남는다.

- 시험 포인트
  - Ethernet은 Link Layer 기술이다.
  - Ethernet 자체는 reliable transfer를 보장하지 않는다.
  - Ethernet frame의 주소는 MAC 주소이다.

---

## 10. Ethernet Switch와 Self-Learning (p61 ~ p71)

- Switch는 Link Layer 장치이다.
- 들어온 Ethernet frame의 MAC 주소를 보고 필요한 output link로 전달한다.
- host 입장에서는 switch 존재를 의식하지 않으므로 transparent하다.

### Switch의 장점

- 여러 링크에서 동시에 전송 가능하다.
- 각 링크를 분리하여 collision domain을 줄인다.
- frame을 모든 곳에 뿌리지 않고 필요한 곳으로만 보낼 수 있다.

```text
A ----\
B ----- [Switch] ----- C
D ----/       \
              E

A→C와 D→E가 동시에 가능할 수 있음
```

### Switch Forwarding Table

```text
MAC address              Interface     TTL
AA-AA-AA-AA-AA-AA        1             ...
BB-BB-BB-BB-BB-BB        2             ...
```

- Switch는 table을 사람이 처음부터 다 입력하지 않아도 스스로 학습한다.

### Self-Learning 원리

```text
frame이 switch의 interface 1로 들어옴
source MAC = A

switch는 기록:
A는 interface 1 방향에 있다.
```

- 즉, switch는 **source MAC 주소**를 보고 학습한다.
- destination MAC 주소는 forwarding 여부를 판단하는 데 사용한다.

### Frame Filtering / Forwarding 규칙

```text
frame 수신 시
1. source MAC과 들어온 interface 기록
2. destination MAC을 switch table에서 검색
3. destination을 알고 있으면
   - 같은 interface 쪽이면 버림(filtering)
   - 다른 interface 쪽이면 그쪽으로 전달(forwarding)
4. destination을 모르면
   - 들어온 포트를 제외하고 flooding
```

- Filtering
  - 목적지가 같은 segment에 있으면 굳이 전달하지 않음.
- Forwarding
  - 목적지가 다른 interface에 있으면 해당 interface로 전달.
- Flooding
  - 목적지를 모르면 일단 여러 포트로 뿌려서 찾음.

### Switch vs Router

| 구분 | Switch | Router |
|---|---|---|
| 계층 | Link Layer, L2 | Network Layer, L3 |
| 기준 주소 | MAC address | IP address |
| 주 역할 | 같은 LAN 내부 frame 전달 | subnet/network 간 datagram 전달 |
| 학습/계산 | self-learning MAC table | routing table / forwarding table |
| broadcast 처리 | 같은 broadcast domain 내 확산 가능 | broadcast domain 분리 |

- 시험 포인트
  - Switch는 source MAC으로 학습한다.
  - 모르는 destination은 flooding한다.
  - Router는 IP 기반, switch는 MAC 기반이다.

---

## 11. VLAN, Virtual LAN (p73 ~ p78)

- VLAN은 하나의 물리적 LAN/switch를 여러 논리적 LAN처럼 나누는 기술이다.
- 물리적으로 같은 switch에 꽂혀 있어도 논리적으로 다른 LAN에 속하게 만들 수 있다.

```text
하나의 물리 switch

ports 1~8   → EE VLAN
ports 9~15  → CS VLAN

물리 장비는 하나지만,
논리적으로는 두 개의 LAN처럼 동작
```

- VLAN이 필요한 이유
  - LAN 규모가 커지면 broadcast traffic이 너무 커진다.
  - 사용자가 사무실을 옮겨도 논리 소속은 유지하고 싶다.
  - 부서별/용도별 traffic isolation이 필요하다.

### Port-based VLAN

- switch port를 기준으로 VLAN을 나눈다.
- 예: port 1~8은 EE, port 9~15는 CS.
- 같은 VLAN 내부 traffic만 직접 전달된다.
- VLAN 사이 통신은 router 또는 L3 switch가 필요하다.

### Trunk Port와 802.1Q

- 여러 switch에 걸쳐 VLAN을 유지하려면 switch 사이 링크에 VLAN ID 정보가 필요하다.
- 이때 trunk port가 여러 VLAN의 frame을 함께 운반한다.
- 802.1Q는 Ethernet frame에 VLAN tag를 추가한다.

```text
[Switch 1] == trunk == [Switch 2]
    |                    |
  VLAN 10              VLAN 10
  VLAN 20              VLAN 20

trunk 링크 frame에는 VLAN ID가 붙어야 함
```

- 802.1Q tag에는 VLAN ID와 priority 정보가 들어간다.
- tag가 추가되면 CRC도 다시 계산해야 한다.

- 시험 포인트
  - VLAN = 물리 LAN을 논리 LAN으로 분리.
  - traffic isolation이 핵심이다.
  - VLAN 간 통신은 routing이 필요하다.
  - trunk port는 여러 VLAN traffic을 운반한다.
  - 802.1Q는 VLAN tag를 붙이는 표준이다.

---

## 12. MPLS, Multiprotocol Label Switching (p80 ~ p85)

- MPLS는 IP forwarding을 빠르고 유연하게 하기 위해 label을 사용하는 기술이다.
- 일반 IP forwarding은 destination IP prefix를 보고 longest prefix matching을 한다.
- MPLS는 짧고 고정 길이인 label을 보고 forwarding한다.

```text
일반 IP forwarding
destination IP → longest prefix match → output interface

MPLS forwarding
label → MPLS forwarding table → output interface
```

- MPLS frame 감각

```text
[Ethernet header][MPLS header][IP header][payload]

MPLS header 안에는 label, Exp, S, TTL 등이 들어감
IP datagram은 여전히 내부에 존재
```

- MPLS capable router는 label-switched router라고도 한다.
- forwarding table은 IP forwarding table과 별도로 존재한다.

### MPLS가 주는 유연성

- IP routing은 보통 목적지 주소 중심으로 경로가 정해진다.
- MPLS는 source/destination, traffic class 등 더 다양한 기준으로 경로를 다르게 정할 수 있다.
- 그래서 traffic engineering에 유리하다.

```text
같은 목적지 A로 가더라도
flow 1 → path X
flow 2 → path Y

목적지 하나만 보는 IP보다 경로 제어가 유연
```

- 장애 시 빠른 우회 경로를 미리 계산해 둘 수 있다.
- OSPF, IS-IS 같은 link-state protocol을 확장해 MPLS에 필요한 정보를 전달할 수 있다.
- RSVP-TE는 MPLS 경로 설정에 쓰이는 signaling protocol이다.

- 시험 포인트
  - MPLS는 IP 주소를 없애는 것이 아니라, IP datagram 앞에 label 기반 forwarding 계층을 덧붙이는 느낌이다.
  - 목적은 빠른 lookup과 traffic engineering이다.
  - label-switched router는 IP 주소가 아니라 label을 보고 넘긴다.

---

## 13. Data Center Networking (p87 ~ p92)

- Data center network는 수만~수십만 대의 host가 가까운 공간에 밀집된 네트워크이다.
- 예: e-business, content server, search engine, data mining 서비스.
- 핵심 문제는 많은 요청을 빠르고 안정적으로 처리하는 것이다.

- 주요 과제
  - 대규모 client 요청 처리
  - reliability 유지
  - load balancing
  - network/data/processing bottleneck 회피
  - 여러 application 간 자원 분배

### Data Center 구성 요소

```text
[Server blades]
     ↓
[Top-of-Rack Switch, TOR]
     ↓
[Tier-2 switches]
     ↓
[Core / Border routers]
     ↓
Internet / external network
```

- Server rack
  - 여러 server blade가 들어 있는 묶음.
- TOR switch
  - rack마다 위치하는 switch.
  - rack 안 server들을 상위 network와 연결.
- Tier switches
  - 여러 rack을 묶고, 다중 경로를 제공.

### Multipath와 Load Balancing

- Data center는 하나의 경로에만 의존하지 않는다.
- 여러 경로를 두어 부하를 분산하고 장애에 대응한다.

```text
Server A
   ├── path 1 ──┐
   ├── path 2 ──┼── Server B
   └── path 3 ──┘

경로가 여러 개면 병목과 장애에 더 강함
```

- 관련 기술 감각
  - RoCE: Ethernet 위에서 RDMA를 지원하는 기술.
  - ECN: 혼잡을 명시적으로 알리는 표시 방식.
  - DCTCP/DCQCN: data center 환경에서 혼잡 제어에 사용되는 흐름.
  - SDN: data center 내부/조직 간 관리에 널리 사용.

- 시험 포인트
  - Data center network는 단순 LAN이 아니라 대규모, 다중 경로, 부하 분산, 혼잡 관리가 중요하다.
  - TOR switch와 rack 구조를 이해해야 한다.

---

## 14. A Day in the Life of a Web Request (p94 ~ p101)

- 이 부분은 전체 protocol stack을 종합하는 예시이다.
- 학생 노트북이 캠퍼스 네트워크에 붙고, 웹 페이지를 요청하는 흐름이다.
- 핵심은 “웹 요청 하나”가 여러 계층 프로토콜을 연쇄적으로 사용한다는 점이다.

```text
노트북 연결
  ↓ DHCP
IP 주소 / gateway / DNS 서버 주소 획득
  ↓ ARP
first-hop router의 MAC 주소 확인
  ↓ DNS
www 주소를 IP 주소로 변환
  ↓ TCP
웹 서버와 연결 설정
  ↓ HTTP
웹 페이지 요청/응답
  ↓ Ethernet/WiFi/IP
각 링크와 네트워크를 따라 실제 전달
```

### 1단계: DHCP로 네트워크 설정 얻기

- 새로 연결된 laptop은 처음에 자기 IP 주소를 모른다.
- DHCP를 통해 다음 정보를 얻는다.
  - 자신의 IP 주소
  - first-hop router 주소
  - DNS server 주소
- DHCP 메시지는 UDP, IP, Ethernet frame에 담겨 전달된다.

```text
Laptop(DHCP client) → DHCP server
"나 이 네트워크에서 쓸 주소가 필요해"
```

### 2단계: ARP로 Router MAC 찾기

- 외부로 나가려면 first-hop router에게 frame을 보내야 한다.
- IP 주소만으로는 Ethernet frame을 보낼 수 없으므로 router의 MAC 주소가 필요하다.
- ARP request/reply로 router MAC을 얻는다.

### 3단계: DNS로 서버 IP 찾기

- 사용자는 `www...` 같은 이름을 입력하지만, 실제 전송에는 IP 주소가 필요하다.
- DNS query를 보내 web server의 IP 주소를 얻는다.

```text
이름: www.example.com
   ↓ DNS
IP: 93.184.216.34 같은 주소
```

### 4단계: TCP 연결 설정

- HTTP 요청을 보내기 전에 TCP 연결을 맺는다.
- SYN, SYN-ACK, ACK의 3-way handshake가 일어난다.

### 5단계: HTTP request/reply

- TCP 연결 위에서 HTTP request가 전송된다.
- server는 HTTP response로 HTML object 등을 돌려준다.
- 이 모든 데이터는 각 링크를 지날 때마다 frame으로 캡슐화된다.

- 시험 포인트
  - DHCP → ARP → DNS → TCP → HTTP 순서 감각을 잡아야 한다.
  - DNS도 결국 UDP/IP/Ethernet 위에 실린다.
  - HTTP도 TCP/IP/Link Layer 위에 실린다.
  - 계층은 분리되어 있지만 실제 통신에서는 계층들이 함께 연쇄 동작한다.

---

## 마지막. 헷갈리는 비교 정리

| 비교 항목 | A | B | 핵심 차이 |
|---|---|---|---|
| Network Layer | host-to-host 경로 | Link Layer | adjacent node 사이 전달 |
| Datagram | Network Layer PDU | Frame | Link Layer PDU |
| IP 주소 | 네트워크 간 라우팅 | MAC 주소 | 같은 LAN 안의 interface 식별 |
| ARP request | broadcast | ARP reply | 보통 unicast |
| TDMA | 시간 분할 | FDMA | 주파수 분할 |
| Slotted ALOHA | slot 시작 때 전송 | Pure ALOHA | 생기면 즉시 전송 |
| ALOHA | 충돌 후 재시도 | CSMA | 먼저 듣고 전송 |
| CSMA | 충돌 가능 | CSMA/CD | 충돌 감지 후 중단 가능 |
| Polling | master가 순서 부여 | Token passing | token 가진 노드가 전송 |
| Ethernet | connectionless/unreliable | TCP | connection-oriented/reliable |
| Switch | MAC 기반 L2 장치 | Router | IP 기반 L3 장치 |
| Flooding | 목적지 모를 때 퍼뜨림 | Forwarding | 목적지 포트로 선택 전달 |
| VLAN | 논리 LAN 분리 | Subnet | IP 네트워크 분리 |
| Access port | 특정 VLAN 소속 | Trunk port | 여러 VLAN frame 운반 |
| IP forwarding | prefix 기반 | MPLS forwarding | label 기반 |
| 일반 LAN | 비교적 소규모 | Data center network | 대규모, 다중 경로, 부하 분산 중요 |

---

## 마지막. 핵심 도식 모음

### Link Layer 캡슐화

```text
IP datagram
    ↓
[Link Header | IP datagram | Link Trailer]
    ↓
Frame
```

### 같은 Subnet에서 전송

```text
A -- Ethernet frame --> B

IP dst = B IP
MAC dst = B MAC
```

### 다른 Subnet으로 전송

```text
A --frame--> Router --frame--> B

A→Router: MAC dst = Router MAC
Router→B: MAC dst = B MAC
IP dst는 계속 B IP
```

### Switch 학습

```text
frame input port 2
source MAC = A

switch table:
A → port 2
```

### VLAN

```text
Physical switch 하나

ports 1~8   = VLAN 10
ports 9~15  = VLAN 20

서로 다른 VLAN은 직접 L2 통신 X
필요하면 routing 필요
```

### 웹 요청 전체 흐름

```text
DHCP → ARP → DNS → TCP → HTTP
주소 받기 → MAC 찾기 → IP 찾기 → 연결 → 웹 요청
```

---

## 마지막. 미니 용어 정리

- Node: host와 router를 모두 포함하는 통신 장치.
- Link: 인접 노드를 연결하는 통신 채널.
- Frame: Link Layer의 데이터 단위.
- NIC: Network Interface Card, Ethernet/WiFi 통신을 담당하는 장치.
- EDC: Error Detection and Correction bits, 오류 검출/수정용 비트.
- CRC: Cyclic Redundancy Check, 강력한 오류 검출 방식.
- MAC: Multiple Access Control 또는 Media Access Control 문맥에서 사용. 여기서는 공유 채널 접근 제어와 MAC 주소 모두 조심해서 구분.
- MAC address: LAN 안에서 interface를 식별하는 48-bit 주소.
- ARP: IP 주소를 MAC 주소로 변환하기 위한 프로토콜.
- CSMA: 전송 전 채널을 먼저 듣는 random access 방식.
- CSMA/CD: 충돌 감지 기능을 포함한 CSMA.
- Backoff: 충돌 후 무작위 시간만큼 기다렸다가 재시도하는 방식.
- Ethernet: 대표적인 유선 LAN 기술.
- Switch: MAC 주소 기반으로 frame을 전달하는 Link Layer 장치.
- VLAN: 물리 LAN을 여러 논리 LAN으로 나누는 기술.
- Trunk: 여러 VLAN traffic을 switch 사이에서 운반하는 링크.
- 802.1Q: VLAN tag를 Ethernet frame에 붙이는 표준.
- MPLS: label 기반으로 빠르고 유연하게 packet을 전달하는 기술.
- TOR Switch: Top-of-Rack switch, 데이터센터 rack 단위 switch.
- RoCE: RDMA over Converged Ethernet.
- ECN: Explicit Congestion Notification, 혼잡 표시 기능.

---

## 마지막 체크리스트

- [ ] Link Layer가 “인접 노드 간 frame 전달”이라는 점을 설명할 수 있는가?
- [ ] IP 주소와 MAC 주소의 역할 차이를 말할 수 있는가?
- [ ] 같은 subnet과 다른 subnet 전송 시 MAC 주소가 어떻게 달라지는지 설명할 수 있는가?
- [ ] ARP request/reply 흐름을 그릴 수 있는가?
- [ ] parity, checksum, CRC의 차이를 구분할 수 있는가?
- [ ] TDMA, FDMA, ALOHA, CSMA/CD의 차이를 비교할 수 있는가?
- [ ] Slotted ALOHA 최대 효율 37%, Pure ALOHA 최대 효율 18%의 의미를 설명할 수 있는가?
- [ ] Ethernet frame 구조의 주요 필드를 말할 수 있는가?
- [ ] Switch가 source MAC으로 self-learning한다는 점을 설명할 수 있는가?
- [ ] destination을 모를 때 flooding한다는 점을 알고 있는가?
- [ ] VLAN이 왜 필요한지, trunk와 802.1Q가 무엇인지 설명할 수 있는가?
- [ ] MPLS가 label 기반 forwarding이라는 점을 설명할 수 있는가?
- [ ] Data center network에서 다중 경로와 부하 분산이 중요한 이유를 설명할 수 있는가?
- [ ] 웹 요청 하나가 DHCP, ARP, DNS, TCP, HTTP를 거친다는 흐름을 순서대로 말할 수 있는가?

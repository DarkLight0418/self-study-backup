# Chapter 6 Link Layer and LANs 핵심 전사 노트

> 목적: 바로 필기 + 시험 직전 복습  
> 형식: 핵심어 / 화살표 / 표 / 암기 문장 중심  
> 참고: Chapter 6 교안 중심 정리. 별도 Chapter 6 손필기 메모는 제공되지 않아 교안 흐름을 v3 템플릿 형식으로 압축함.

---

## 0. 먼저 외울 핵심 문장

- Link Layer = 인접 노드끼리 datagram을 frame으로 전달하는 계층.
- Node = host + router.
- Link = 인접 노드를 연결하는 물리/무선/LAN 통신 채널.
- Frame = Link Layer packet, IP datagram을 캡슐화함.
- 링크 계층 주소 = MAC address, 네트워크 계층 주소 = IP address.
- IP 주소는 목적지 네트워크까지 가는 주소, MAC 주소는 같은 링크 안에서 다음 홉까지 가는 주소.
- 같은 subnet 안에서는 ARP로 목적지 MAC을 찾는다.
- 다른 subnet으로 갈 때는 목적지 IP는 최종 목적지, 목적지 MAC은 first-hop router의 MAC.
- Error detection은 오류를 발견하는 것, correction은 재전송 없이 고치는 것.
- CRC는 Ethernet/WiFi에서 널리 쓰이는 강한 오류 검출 방식.
- Multiple Access Protocol = 공유 채널을 여러 노드가 어떻게 나눠 쓸지 정하는 규칙.
- TDMA/FDMA = 채널을 미리 나눔, 충돌은 적지만 빈 슬롯/대역 낭비 가능.
- ALOHA = 그냥 보내고 충돌 나면 확률적으로 재전송.
- CSMA = 보내기 전 듣기, CSMA/CD = 충돌 감지 후 중단.
- Switch = MAC 주소를 보고 frame을 전달하는 Link Layer 장비.
- Switch는 self-learning으로 forwarding table을 만든다.
- Flooding = 목적지 위치를 모를 때 들어온 포트를 제외하고 모두 전송.
- VLAN = 물리적으로 하나의 LAN을 논리적으로 여러 LAN처럼 나누는 기술.
- Trunk port = 여러 VLAN의 frame을 switch 사이에서 운반하는 포트.
- MPLS = IP 주소 대신 label로 빠르게 forwarding하는 방식.
- Data center network는 다중 경로, load balancing, SDN, 고속 Ethernet이 중요하다.
- Web request 하나에도 DHCP, ARP, DNS, TCP, HTTP, Ethernet, IP가 함께 동작한다.

---

## 1. Link Layer 큰 그림 (p6-2 ~ p6-9)

- 핵심: Link Layer는 `한 홉` 단위 전달 계층.
- 구조: `IP datagram → frame으로 포장 → 인접 node로 전달 → datagram 추출`
- 암기: `Network Layer = end-to-end 방향`, `Link Layer = hop-to-hop 실제 전달`
- 주의: 같은 datagram도 이동하는 링크마다 다른 link protocol 사용 가능.
- 시험: node/link/frame/NIC/MAC address 구분.

```text
[Host A] --link1-- [Router] --link2-- [Host B]
  WiFi              Ethernet 가능

IP datagram은 끝까지 가지만,
Frame은 각 링크마다 새로 만들어질 수 있음.
```

- 주요 서비스:
  - framing = datagram에 header/trailer 붙여 frame 생성
  - link access = 공유 매체에서 누가 보낼지 결정
  - reliable delivery = 인접 노드 사이 신뢰성 제공 가능
  - flow control = 인접 송수신 속도 조절
  - error detection/correction = bit error 발견/수정
  - half duplex = 양방향 가능하지만 동시에 X
  - full duplex = 양방향 동시 가능

---

## 2. Link Layer 구현 위치: NIC (p6-8 ~ p6-9)

- 핵심: Link Layer는 주로 NIC(Network Interface Card)에서 구현.
- 구조: `CPU/Memory → Host bus → NIC controller → Physical link`
- 암기: `NIC = Link + Physical 계층 담당 장치`
- 주의: 소프트웨어만이 아니라 hardware + firmware + software 조합.
- 시험: 송신/수신 시 NIC 역할.

```text
송신:
[Network Layer datagram]
      ↓
[NIC: frame 생성 + 오류검출 비트 추가]
      ↓
[Physical link]

수신:
[Frame 수신]
      ↓
[NIC: 오류 확인 + datagram 추출]
      ↓
[Network Layer로 전달]
```

---

## 3. Error Detection / Correction (p6-11 ~ p6-15)

- 핵심: 전송 중 bit flip을 발견하거나 고치는 기술.
- 구조: `Data(D) + EDC → 전송 → 수신자가 검사`
- 암기: `EDC가 클수록 검출/수정 능력 증가, 하지만 오버헤드 증가`
- 주의: 오류 검출은 100% 완벽하지 않음.
- 시험: parity / checksum / CRC 비교.

| 방식 | 핵심 | 장점 | 주의 |
|---|---|---|---|
| Parity | 1의 개수를 짝수/홀수로 맞춤 | 단순 | 다중 오류에 약함 |
| 2D Parity | 행/열 parity 사용 | single-bit error 위치 수정 가능 | 복잡도 증가 |
| Internet checksum | 16-bit 단위 합산 | UDP 등에서 사용 | 보호 약함 |
| CRC | generator G로 나눗셈 | burst error 검출 강함 | 계산 구조 이해 필요 |

```text
수식: <D,R> = D · 2^r XOR R = nG
의미: 데이터 D 뒤에 r비트 R을 붙여 G로 나누어 떨어지게 만듦
암기: 수신자는 <D,R>을 G로 나눠 나머지가 0이면 통과, 아니면 오류
```

- CRC 흐름:

```text
[Sender]
D와 G가 주어짐
→ D 뒤에 r개의 0 추가
→ G로 나눈 나머지 R 계산
→ <D,R> 전송

[Receiver]
<D,R>을 G로 나눔
→ 나머지 0: 오류 없음으로 판단
→ 나머지 ≠ 0: 오류 검출
```

---

## 4. Multiple Access Protocol 개요 (p6-17 ~ p6-20)

- 핵심: 공유 채널에서 여러 노드가 언제 보낼지 정하는 규칙.
- 구조: `공유 채널 + 동시 전송 → interference/collision → MAC protocol 필요`
- 암기: `MAC = Medium Access Control`
- 주의: channel sharing을 위한 통신도 같은 채널을 사용해야 함.
- 시험: ideal MAC protocol 조건과 3분류.

- 이상적인 MAC 조건:
  - 한 노드만 전송 → rate R 전체 사용
  - M개 노드 전송 → 평균 R/M씩 공정 사용
  - 완전 분산형 → 특별한 조정 노드 없음
  - 단순해야 함

```text
MAC Protocol 3분류

1) Channel Partitioning
   → 시간/주파수/코드로 미리 나눔

2) Random Access
   → 그냥 보내고 충돌 나면 회복

3) Taking Turns
   → 순서를 정해 돌아가며 전송
```

---

## 5. Channel Partitioning: TDMA / FDMA (p6-21 ~ p6-22)

- 핵심: 채널을 작은 조각으로 나누고 각 노드에 독점 배정.
- 구조: `Channel → time slot 또는 frequency band → station별 할당`
- 암기: `TDMA = 시간 나눔`, `FDMA = 주파수 나눔`
- 주의: 보낼 데이터가 없어도 할당된 slot/band가 비면 낭비.
- 시험: random access와 장단점 비교.

| 구분 | TDMA | FDMA |
|---|---|---|
| 나누는 기준 | 시간 slot | 주파수 band |
| 전송 방식 | 자기 차례 slot에 전송 | 자기 band에서 전송 |
| 장점 | 충돌 적음, 공정 | 충돌 적음, 동시 전송 가능 |
| 단점 | 빈 slot 낭비 | 빈 주파수 대역 낭비 |

```text
TDMA 예시
Round: | 1 | 2 | 3 | 4 | 5 | 6 |
사용 : | O | X | O | O | X | X |

FDMA 예시
Frequency: [band1][band2][band3][band4]
Station  :   1      2      3      4
```

---

## 6. Random Access: ALOHA 계열 (p6-23 ~ p6-27, p6-105)

- 핵심: 채널을 나누지 않고, 각 노드가 보낼 게 있으면 전송.
- 구조: `전송 → 충돌 가능 → 지연 후 재전송`
- 암기: `Random Access = 충돌 허용 + 충돌 회복`
- 주의: 부하가 커지면 충돌이 급증.
- 시험: Slotted ALOHA 효율 37%, Pure ALOHA 효율 18%.

### Slotted ALOHA

- 핵심: 시간을 slot으로 나누고 slot 시작 시점에만 전송.
- 구조: `slot 시작 → frame 전송 → 성공/충돌/빈 slot`
- 암기: `동기화 필요, 최대 효율 1/e ≈ 0.37`
- 주의: 성공 slot보다 collision/empty slot 낭비가 큼.
- 시험: 확률식 자주 나옴.

```text
수식: P(any success) = Np(1-p)^(N-1)
의미: N개 노드 중 정확히 1개만 전송해야 성공
암기: 최대 효율 = 1/e ≈ 0.37
```

### Pure ALOHA

- 핵심: slot 없이 frame이 생기면 즉시 전송.
- 구조: `frame 도착 → 즉시 전송 → 충돌 가능 구간 증가`
- 암기: `동기화 없음, 효율 1/(2e) ≈ 0.18`
- 주의: Slotted ALOHA보다 충돌 가능 구간이 2배.
- 시험: Pure ALOHA가 더 단순하지만 효율이 더 낮음.

```text
Pure ALOHA 충돌 구간

       t0
-------|--------
[t0-1, t0]에 시작한 frame도 충돌 가능
[t0, t0+1]에 시작한 frame도 충돌 가능
```

---

## 7. CSMA / CSMA-CD (p6-28 ~ p6-32)

- 핵심: CSMA는 `듣고 나서 보내는` random access 방식.
- 구조: `channel sense → idle이면 전송 / busy면 대기`
- 암기: `CSMA = 말 끊지 않기`, `CSMA/CD = 충돌 감지하면 말 멈추기`
- 주의: propagation delay 때문에 듣고 보내도 충돌 가능.
- 시험: CSMA/CD 알고리즘과 efficiency 식.

```text
CSMA 흐름
[보낼 frame 있음]
      ↓
[채널 감지]
 ┌────┴────┐
idle      busy
 ↓         ↓
전송       대기
```

- CSMA/CD 알고리즘:
  - 1) NIC가 datagram을 받아 frame 생성
  - 2) channel idle → 전송 시작
  - 3) channel busy → idle 될 때까지 대기
  - 4) 충돌 없이 끝까지 전송 → 완료
  - 5) 전송 중 충돌 감지 → 중단 + jam signal
  - 6) binary exponential backoff 후 재시도

```text
수식: efficiency = 1 / (1 + 5 · Tprop / Ttrans)
의미: 전파 지연이 작고 frame 전송 시간이 길수록 효율 증가
암기: Tprop ↓ 또는 Ttrans ↑ → efficiency → 1
```

---

## 8. Taking Turns와 Cable Access (p6-33 ~ p6-38)

- 핵심: 순서를 정해 충돌 없이 전송하려는 방식.
- 구조: `polling 또는 token passing → 차례가 온 노드가 전송`
- 암기: `Taking Turns = partitioning의 공정성 + random access의 효율을 노림`
- 주의: polling/token 자체 오버헤드와 장애 지점 존재.
- 시험: channel partitioning/random access/taking turns 비교.

| 방식 | 구조 | 장점 | 주의 |
|---|---|---|---|
| Polling | master가 slave에게 차례 부여 | 충돌 감소 | master 장애, polling overhead |
| Token Passing | token 가진 노드만 전송 | 분산적 순서 제어 | token 손실/오버헤드 |
| DOCSIS | downstream FDM + upstream TDM/random | cable access에 적합 | request slot 경쟁 발생 |

```text
Polling
[Master] → poll → [Node1] → data
[Master] → poll → [Node2] → data

Token Passing
[Node1] --T--> [Node2] --T--> [Node3]
토큰 가진 노드만 전송 가능
```

---

## 9. MAC Address와 ARP (p6-40 ~ p6-46)

- 핵심: MAC address는 같은 LAN 안에서 frame을 보낼 때 쓰는 주소.
- 구조: `IP 주소를 알고 있음 → ARP로 MAC 주소 확인 → Ethernet frame 전송`
- 암기: `IP = 우편 주소`, `MAC = 주민등록번호 같은 NIC 고유 주소`
- 주의: IP는 subnet에 따라 바뀔 수 있지만 MAC은 flat address라 이동 가능.
- 시험: IP address vs MAC address, ARP 동작 순서.

| 구분 | IP Address | MAC Address |
|---|---|---|
| 계층 | Network Layer | Link Layer |
| 길이 | 32-bit IPv4 | 48-bit 일반적 |
| 역할 | 목적지 네트워크까지 전달 | 같은 링크 안 interface까지 전달 |
| 성격 | subnet 의존 | NIC 중심, flat address |

- ARP Table:

```text
< IP address ; MAC address ; TTL >
TTL = 매핑 정보를 유지하는 시간
```

- ARP 동작:

```text
A가 B에게 보내고 싶음
A는 B의 IP는 알고 있지만 MAC은 모름
      ↓
A가 ARP Query broadcast
Destination MAC = FF-FF-FF-FF-FF-FF
      ↓
LAN의 모든 노드가 받음
      ↓
B만 ARP Reply로 자신의 MAC 전달
      ↓
A가 ARP table에 저장
```

---

## 10. 다른 Subnet으로 보낼 때 주소 변화 (p6-47 ~ p6-52)

- 핵심: IP datagram의 목적지 IP는 최종 목적지로 유지되지만, frame의 MAC 주소는 홉마다 바뀜.
- 구조: `A → first-hop router R → B`
- 암기: `IP는 끝까지, MAC은 다음 홉까지`
- 주의: 다른 subnet 목적지로 보낼 때 frame destination은 B MAC이 아니라 router MAC.
- 시험: IP src/dest와 MAC src/dest 변화 구분.

```text
A에서 R로 보낼 때
IP src  = A IP
IP dest = B IP
MAC src = A MAC
MAC dest = R의 A쪽 interface MAC

R에서 B로 보낼 때
IP src  = A IP
IP dest = B IP
MAC src = R의 B쪽 interface MAC
MAC dest = B MAC
```

```text
[A] -- frame1 --> [Router R] -- frame2 --> [B]
     MAC A→R                   MAC R→B

IP datagram: A IP → B IP  (변하지 않음)
Frame: 링크마다 새로 생성됨
```

---

## 11. Ethernet 기본 구조 (p6-54 ~ p6-59)

- 핵심: Ethernet은 대표적인 wired LAN 기술.
- 구조: `IP datagram → Ethernet frame → NIC/Link → 수신 NIC`
- 암기: `Ethernet = connectionless + unreliable + CSMA/CD 기반`
- 주의: Ethernet 자체는 ACK/NAK 없음. 손실 복구는 상위 계층 TCP 등이 담당.
- 시험: Ethernet frame field와 switched topology.

```text
Ethernet Frame
[preamble][dest MAC][source MAC][type][data][CRC]
```

- 필드 암기:
  - preamble = sender/receiver clock 동기화
  - dest/source address = 각각 6 byte MAC 주소
  - type = 상위 계층 프로토콜 표시, 예: IP
  - data = payload, 보통 IP datagram
  - CRC = 오류 검출, 오류 발견 시 frame drop

| 구조 | Bus Ethernet | Switched Ethernet |
|---|---|---|
| 형태 | 하나의 공유 cable | switch 중심 star 구조 |
| 충돌 | 같은 collision domain | 각 link가 별도 collision domain |
| 현재 | 과거 방식 | 현재 지배적 방식 |

---

## 12. Ethernet Switch와 Self-Learning (p6-61 ~ p6-71)

- 핵심: Switch는 MAC 주소 기반으로 frame을 선택적으로 전달하는 Link Layer 장비.
- 구조: `frame 수신 → source MAC 학습 → destination MAC 조회 → forward/drop/flood`
- 암기: `Switch = plug-and-play + self-learning + transparent`
- 주의: switch는 IP header가 아니라 link-layer header를 본다.
- 시험: filtering/forwarding 규칙, flooding, switch vs router.

- Switch 특징:
  - store-and-forward 방식
  - host는 switch 존재를 의식하지 않음 → transparent
  - 설정 없이 동작 가능 → plug-and-play
  - 여러 link에서 동시 전송 가능
  - full-duplex link에서는 충돌 없음

```text
Switch table entry
< MAC address ; interface ; TTL >
```

- Self-learning:

```text
Frame 도착
Source MAC = A
Incoming interface = 1
      ↓
Switch table에 기록
A → interface 1
```

- Forwarding 규칙:

```text
1. source MAC과 들어온 interface 기록
2. destination MAC을 switch table에서 검색
3. destination이 같은 interface 쪽이면 drop
4. destination 위치를 알면 해당 interface로 forward
5. destination 위치를 모르면 flood
```

| 구분 | Switch | Router |
|---|---|---|
| 계층 | Link Layer | Network Layer |
| 보는 주소 | MAC address | IP address |
| 테이블 생성 | self-learning, flooding | routing algorithm/protocol |
| 단위 | frame | datagram |
| 역할 | LAN 내부 전달 | subnet/network 사이 전달 |

---

## 13. VLAN: Virtual LAN (p6-73 ~ p6-78)

- 핵심: VLAN은 하나의 물리 LAN을 여러 논리 LAN처럼 나누는 기술.
- 구조: `Physical switch → port group → 여러 broadcast domain`
- 암기: `VLAN = 물리적 위치와 논리적 소속 분리`
- 주의: VLAN 간 통신은 routing 필요.
- 시험: VLAN 필요성, trunk port, 802.1Q frame.

- VLAN이 필요한 이유:
  - LAN 규모 커짐 → ARP/DHCP/unknown MAC broadcast가 전체 LAN에 퍼짐
  - 보안/프라이버시/효율 문제 발생
  - 사용자가 물리적으로 이동해도 논리적 부서는 유지하고 싶음

```text
하나의 switch 내부
ports 1-8   = EE VLAN
ports 9-15  = CS VLAN

서로 다른 VLAN은 같은 switch 안에 있어도 별도 LAN처럼 동작
```

- Trunk port:

```text
[Switch 1] == trunk == [Switch 2]
   VLAN 10 frame
   VLAN 20 frame
   VLAN 30 frame

trunk는 여러 VLAN frame을 함께 운반
→ frame에 VLAN ID 필요
```

- 802.1Q:
  - Ethernet frame에 VLAN tag 추가
  - TPID = 0x8100
  - TCI 안에 12-bit VLAN ID, priority field 포함
  - CRC는 tag 추가 후 재계산

---

## 14. MPLS: Multiprotocol Label Switching (p6-80 ~ p6-85)

- 핵심: MPLS는 IP 주소 대신 fixed-length label을 보고 빠르게 forwarding.
- 구조: `IP packet + MPLS label → label-switched router → label 기반 전달`
- 암기: `MPLS = IP는 유지, forwarding은 label로 빠르게`
- 주의: MPLS forwarding table은 IP forwarding table과 별도.
- 시험: MPLS vs IP routing, label swapping, traffic engineering.

```text
MPLS Header
[label][Exp][S][TTL]
 20b   3b  1b 5b
```

- MPLS 장점:
  - fixed-length label → lookup 빠름
  - 목적지뿐 아니라 source 등 다른 field 기반 경로 설정 가능
  - traffic engineering 가능
  - link failure 시 pre-computed backup path로 빠른 reroute 가능

| 구분 | IP Routing | MPLS |
|---|---|---|
| forwarding 기준 | destination IP | label |
| 경로 선택 | 주로 목적지 주소 | source/destination/정책 가능 |
| 장점 | 범용성 | 빠른 lookup, 유연한 경로 제어 |
| 특징 | longest prefix matching | virtual circuit 아이디어 차용 |

---

## 15. Data Center Networking (p6-87 ~ p6-92)

- 핵심: Data center network는 대량 host와 대량 client 요청을 고속/고가용성으로 처리하는 구조.
- 구조: `Border router → Tier switch → TOR switch → server rack`
- 암기: `TOR = Top of Rack switch`
- 주의: 병목은 compute뿐 아니라 network/load/data 위치에서도 발생.
- 시험: datacenter network 요소, multipath, load balancer 역할.

```text
[Internet]
    ↓
[Border Router]
    ↓
[Tier-1 Switch]
    ↓
[Tier-2 Switch]
    ↓
[TOR Switch]
    ↓
[Server Rack]
```

- 핵심 요소:
  - border router = 외부 네트워크 연결
  - tier-1/tier-2 switch = rack 사이 연결
  - TOR switch = rack별 switch
  - server rack = 다수 server blade

- 주요 기술:
  - multipath = rack 사이 여러 경로로 throughput/reliability 증가
  - load balancer = 외부 요청을 내부 server로 분산
  - RoCE = RDMA over Converged Ethernet
  - ECN/DCTCP/DCQCN = data center 혼잡 제어 관련
  - SDN = datacenter 내부/조직 간 관리에 널리 사용

---

## 16. A Day in the Life of a Web Request (p6-94 ~ p6-101)

- 핵심: 웹 페이지 요청 하나에도 여러 계층/프로토콜이 연쇄적으로 동작.
- 구조: `DHCP → ARP → DNS → TCP → HTTP → Web page`
- 암기: `주소 받기 → MAC 찾기 → IP 찾기 → TCP 연결 → HTTP 요청`
- 주의: 단순한 웹 요청도 application/transport/network/link 계층이 모두 관여.
- 시험: 순서와 캡슐화/역캡슐화 흐름.

```text
1. DHCP
노트북이 네트워크 접속
→ 내 IP, first-hop router IP, DNS server IP 획득

2. ARP
DNS query를 보내려면 router MAC 필요
→ ARP broadcast
→ router가 ARP reply

3. DNS
www.google.com의 IP 주소 질의
→ DNS server가 IP address 응답

4. TCP
web server와 3-way handshake
→ SYN → SYNACK → ACK

5. HTTP
HTTP request 전송
→ HTTP reply 수신
→ web page 표시
```

- 캡슐화 흐름:

```text
Application: HTTP / DNS / DHCP
        ↓
Transport: TCP or UDP
        ↓
Network: IP datagram
        ↓
Link: Ethernet frame
        ↓
Physical: bits
```

---

## 마지막 비교 정리

| 비교 | A | B | 핵심 차이 |
|---|---|---|---|
| Network Layer | end-to-end | Link Layer | hop-to-hop |
| IP Address | 32-bit, network layer | MAC Address | 48-bit, link layer |
| Datagram | IP packet | Frame | link-layer packet |
| Error Detection | 오류 발견 | Error Correction | 오류 수정까지 수행 |
| Parity | 단순 검출 | CRC | 강한 오류 검출 |
| TDMA | 시간 나눔 | FDMA | 주파수 나눔 |
| Slotted ALOHA | slot 동기화 | Pure ALOHA | 즉시 전송, 효율 낮음 |
| ALOHA | 충돌 후 복구 | CSMA | 보내기 전 채널 감지 |
| CSMA | 감지만 함 | CSMA/CD | 충돌 감지 후 중단 |
| Polling | master가 순서 부여 | Token Passing | token 가진 노드 전송 |
| Same Subnet | 목적지 MAC = 상대 host | Different Subnet | 목적지 MAC = router |
| Ethernet | connectionless/unreliable | TCP | connection-oriented/reliable |
| Switch | MAC 기반 전달 | Router | IP 기반 전달 |
| Flooding | 목적지 모를 때 전체 전송 | Forwarding | 목적지 포트만 전송 |
| VLAN | 논리적 LAN 분리 | Physical LAN | 물리 연결 기준 |
| Access Port | 하나의 VLAN 소속 | Trunk Port | 여러 VLAN 운반 |
| IP Routing | destination 기반 | MPLS | label 기반 |
| Data Center | multipath/load balancing | 일반 LAN | 대규모 트래픽/고가용성 중심 |
| DHCP | 내 네트워크 설정 획득 | DNS | 이름을 IP로 변환 |
| ARP | IP → MAC | DNS | Domain → IP |

---

## 최종 체크리스트

- [ ] Link Layer가 `hop-to-hop 전달`임을 설명할 수 있다.
- [ ] Datagram과 Frame의 차이를 말할 수 있다.
- [ ] MAC 주소와 IP 주소의 역할 차이를 설명할 수 있다.
- [ ] 같은 subnet과 다른 subnet에서 MAC 목적지가 어떻게 달라지는지 설명할 수 있다.
- [ ] ARP query/reply 흐름을 순서대로 말할 수 있다.
- [ ] CRC 수식의 의미를 설명할 수 있다.
- [ ] Parity, checksum, CRC를 비교할 수 있다.
- [ ] TDMA, FDMA, Random Access, Taking Turns를 비교할 수 있다.
- [ ] Slotted ALOHA와 Pure ALOHA의 효율 차이를 기억한다.
- [ ] CSMA/CD의 binary exponential backoff 흐름을 설명할 수 있다.
- [ ] Ethernet frame 구조를 그릴 수 있다.
- [ ] Switch의 self-learning과 flooding 조건을 설명할 수 있다.
- [ ] Switch와 Router의 차이를 구분할 수 있다.
- [ ] VLAN의 필요성과 trunk port의 역할을 설명할 수 있다.
- [ ] MPLS가 IP routing과 어떻게 다른지 설명할 수 있다.
- [ ] Data center network에서 load balancer와 multipath의 필요성을 설명할 수 있다.
- [ ] 웹 요청 흐름을 DHCP → ARP → DNS → TCP → HTTP 순서로 설명할 수 있다.

---

## 시험 직전 1분 압축

```text
Link Layer = 인접 노드 간 frame 전달
Frame = datagram + link header/trailer
MAC = 같은 LAN 안에서 다음 interface 찾기
ARP = IP를 MAC으로 바꾸는 LAN 내부 질의
Switch = MAC 기반 self-learning forwarding
Router = IP 기반 subnet 사이 forwarding
VLAN = 물리 LAN을 논리 LAN으로 분리
MPLS = IP 대신 label로 빠른 forwarding
Web request = DHCP → ARP → DNS → TCP → HTTP
```

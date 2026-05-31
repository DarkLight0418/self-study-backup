# Chapter 5 Network Layer: Control Plane 핵심 전사 노트

> 목적: 교안 확인하면서 바로 옮겨 적기
> 형식: 긴 서술 X / 핵심어 + 화살표 + 비교 중심
> 기준: Control Plane → Routing Algorithm → OSPF/BGP → SDN → ICMP → Management

---

## 0. 먼저 외울 핵심 문장

- Data Plane = 패킷을 어느 출력 포트로 보낼지 결정
- Control Plane = 경로를 어떻게 정할지 결정
- Forwarding = 라우터 내부의 즉시 처리
- Routing = 출발지에서 목적지까지의 경로 결정
- Per-router 방식 = 각 라우터가 라우팅 알고리즘 수행
- SDN 방식 = 중앙 컨트롤러가 경로 계산 후 테이블 설치
- Routing Protocol 목표 = 좋은 path 선택
- Good path = 최소 cost / 빠름 / 덜 혼잡한 경로
- Link State = 전체 지도를 알고 최단 경로 계산
- Distance Vector = 이웃이 알려준 거리 정보로 갱신
- Dijkstra = 하나의 출발 노드에서 모든 목적지 최단 경로 계산
- Bellman-Ford = 이웃을 거쳐 목적지로 가는 최소 비용 선택
- Good news travels fast = 비용 감소 정보는 빠르게 퍼짐
- Bad news travels slow = 비용 증가 정보는 천천히 퍼져 count-to-infinity 발생 가능
- AS = 하나의 관리 주체가 운영하는 네트워크 묶음
- OSPF = AS 내부에서 쓰는 Link-State 기반 라우팅
- BGP = AS 사이를 연결하는 inter-domain 라우팅
- BGP는 성능보다 정책이 더 중요
- Hot potato routing = 내 AS 밖으로 최대한 빨리 내보내기
- SDN = Control Plane과 Data Plane을 분리
- ICMP = 네트워크 오류/상태 알림 메시지
- SNMP = 장비 상태 조회/설정용 관리 프로토콜
- NETCONF/YANG = 구조화된 네트워크 설정 관리 방식

---

## 1. Control Plane 큰 그림 (5-2 ~ 5-6)

### 핵심 구조

| 구분 | 의미 | 손필기식 암기 |
|---|---|---|
| Data Plane | 들어온 패킷을 어느 포트로 보낼지 처리 | 지금 이 패킷 어디로? |
| Control Plane | 경로/포워딩 테이블을 어떻게 만들지 결정 | 앞으로 어떤 길로? |

### Forwarding vs Routing

| 구분 | Forwarding | Routing |
|---|---|---|
| 위치 | 라우터 내부 | 네트워크 전체 |
| 단위 | 패킷 1개 | 출발지~목적지 경로 |
| 속도 | 매우 빠름 | 상대적으로 느림 |
| 역할 | 입력 포트 → 출력 포트 | 좋은 path 계산 |

```text
[Control Plane]
   ↓ 경로 계산
[Forwarding Table 생성]
   ↓
[Data Plane]
   ↓
패킷 입력 → 테이블 조회 → 출력 포트 선택
```

### Control Plane 방식 2가지

| 방식 | 구조 | 암기 |
|---|---|---|
| Per-router control | 각 라우터가 직접 알고리즘 수행 | 라우터들이 각자 계산 |
| SDN control | 원격 컨트롤러가 계산 후 설치 | 중앙 두뇌가 계산 |

```text
Per-router:
[R1 알고리즘] ↔ [R2 알고리즘] ↔ [R3 알고리즘]
각 라우터가 정보 교환 + 테이블 계산

SDN:
        [Remote Controller]
          ↓      ↓      ↓
        [R1]    [R2]    [R3]
컨트롤러가 계산 → 라우터에 테이블 설치
```

---

## 2. Routing Protocol과 Graph Abstraction (5-8 ~ 5-10)

### Routing Protocol 목표

- 목적: 출발 host → 목적 host까지 좋은 path 결정
- path = 패킷이 지나가는 라우터들의 순서
- good path = cost가 낮은 경로
- cost 기준 = 홉 수 / 대역폭 / 지연 / 혼잡도 등

### 그래프 모델

```text
G = (N, E)

N = routers 집합
E = links 집합
c(x,y) = x와 y 사이 직접 링크 비용
직접 연결 X → c(x,y) = ∞
```

### 라우팅 알고리즘 분류

| 분류 | 방식 | 핵심 표현 |
|---|---|---|
| Global | 전체 topology + cost를 모두 앎 | 전체 지도 들고 계산 |
| Decentralized | 이웃 정보부터 점진 계산 | 이웃 말 듣고 조금씩 계산 |
| Static | 경로 변화 거의 없음 | 고정 경로 |
| Dynamic | 변화에 따라 갱신 | 상황 따라 재계산 |

### LS vs DV 한 줄 구분

- Link State = 전체 지도 + Dijkstra
- Distance Vector = 이웃 거리표 + Bellman-Ford

---

## 3. Link-State Routing / Dijkstra (5-12 ~ 5-18)

### Link-State 핵심

- 모든 라우터가 전체 topology와 link cost를 앎
- Link-state broadcast로 정보를 퍼뜨림
- 각 라우터가 같은 지도를 보고 자기 기준 최단 경로 계산

```text
모든 라우터가 전체 지도 확보
        ↓
각자 자기 자신을 source로 둠
        ↓
Dijkstra로 최단 경로 트리 계산
        ↓
Forwarding Table 생성
```

### Dijkstra에서 쓰는 기호

| 기호 | 의미 |
|---|---|
| c(x,y) | x-y 직접 링크 비용 |
| D(v) | source에서 v까지 현재 추정 최소 비용 |
| p(v) | v로 가기 직전 predecessor 노드 |
| N' | 최단 경로가 확정된 노드 집합 |

### Dijkstra 절차

```text
1. 시작 노드 u 확정 → N' = {u}
2. u와 직접 연결된 노드의 D값 기록
3. N' 밖에서 D값이 가장 작은 노드 w 선택
4. w를 N'에 추가
5. w를 거쳐 가는 경로가 더 싸면 D(v) 갱신
6. 모든 노드가 N'에 들어올 때까지 반복
```

### 핵심 갱신식

```text
D(v) = min( D(v), D(w) + c(w,v) )

기존 u→v 비용
vs
u→w 확정 비용 + w→v 직접 비용
둘 중 더 작은 값 선택
```

### 표 읽는 법

| 칸 | 보는 방법 |
|---|---|
| N' | 최단 경로 확정 노드 |
| D(v), p(v) | v까지 비용, 직전 노드 |
| 가장 작은 D | 다음 확정 노드 |
| p(v) 추적 | 최단 경로 tree 복원 |

### Dijkstra 주의

- 직접 연결이 없으면 초기 비용은 ∞
- 매 단계에서 가장 작은 D를 먼저 확정
- 한 번 N'에 들어가면 최단 거리 확정
- tie가 있으면 임의 선택 가능
- 복잡도: 기본 O(n²), 개선 시 O(n log n)

### Oscillation

- link cost가 traffic volume에 따라 바뀌면 경로가 흔들릴 수 있음
- 경로 변경 → 트래픽 이동 → cost 변경 → 다시 경로 변경

```text
트래픽 몰림
   ↓
해당 link cost 증가
   ↓
라우팅 경로 변경
   ↓
다른 link로 트래픽 몰림
   ↓
다시 cost 변화
```

---

## 4. Distance Vector / Bellman-Ford (5-20 ~ 5-40)

### Distance Vector 핵심

- 각 라우터는 처음에 이웃까지의 cost만 앎
- 이웃이 자기 distance vector를 보내줌
- 받은 정보를 바탕으로 내 거리표 갱신
- 반복 + 비동기 + 분산 방식

### Bellman-Ford 식

```text
Dx(y) = min_v { c(x,v) + Dv(y) }

Dx(y): x에서 y까지 가는 최소 비용
c(x,v): x에서 이웃 v까지 직접 비용
Dv(y): 이웃 v가 알고 있는 y까지 비용
v: x의 모든 이웃 후보
```

### 손필기식 이해

```text
x가 y로 가고 싶다.
직접 전체 지도를 모른다.
그래서 이웃들에게 물어본다.

"너 y까지 얼마에 갈 수 있어?"

내 비용 = 나→이웃 비용 + 이웃→목적지 비용
그중 가장 싼 이웃을 next hop으로 선택
```

### DV 동작 순서

```text
1. local link cost 변화 감지
또는
1. neighbor로부터 DV 메시지 수신
        ↓
2. Bellman-Ford 식으로 내 DV 재계산
        ↓
3. 내 DV가 바뀌면 이웃에게 알림
        ↓
4. 안 바뀌면 조용히 멈춤
```

### DV 특징

| 특징 | 의미 |
|---|---|
| Iterative | 계속 반복해서 수렴 |
| Asynchronous | 모든 라우터가 동시에 움직일 필요 없음 |
| Distributed | 중앙 관리자 없이 이웃끼리 계산 |
| Self-stopping | 변화 없으면 메시지 전송 멈춤 |

### Link cost 감소: Good news travels fast

```text
비용 감소 발생
   ↓
더 좋은 경로 즉시 발견
   ↓
이웃에게 빠르게 전파
```

- 좋은 소식 = 더 싼 경로 발견
- 보통 빠르게 안정화됨

### Link cost 증가: Bad news travels slow

```text
비용 증가 발생
   ↓
이웃이 예전 정보를 믿음
   ↓
서로를 경유하면 된다고 착각
   ↓
비용이 6, 7, 8, 9... 계속 증가
   ↓
count-to-infinity
```

### Count-to-infinity

- 나쁜 경로 정보가 천천히 수정되는 문제
- 라우팅 loop 발생 가능
- 분산 알고리즘의 대표적 어려움

### Black-holing

- 잘못된 DV 광고로 트래픽이 특정 라우터에 빨려 들어가는 문제
- 예: “내가 모든 곳에 싸게 갈 수 있음”이라고 거짓 광고

### LS vs DV 비교

| 항목 | Link State | Distance Vector |
|---|---|---|
| 정보 범위 | 전체 topology | 이웃의 거리표 |
| 알고리즘 | Dijkstra | Bellman-Ford |
| 계산 위치 | 각 라우터가 전체 지도 계산 | 이웃 정보 기반 갱신 |
| 메시지 | 전체에 link-state broadcast | 이웃과 DV 교환 |
| 장점 | 전체 구조 파악 | 구현 상대적 단순 |
| 단점 | 메시지 복잡도, oscillation | loop, count-to-infinity |
| 오류 영향 | 자기 테이블 위주 | 오류가 주변으로 전파 |

---

## 5. AS / Intra-AS / OSPF (5-42 ~ 5-48)

### 왜 AS가 필요한가

- 인터넷은 flat 구조가 아님
- 모든 라우터를 하나로 관리 불가
- 목적지가 너무 많아 routing table이 커짐
- 네트워크마다 관리 정책이 다름

### AS 의미

```text
AS = Autonomous System
= 하나의 관리 주체가 운영하는 네트워크 영역
= domain이라고도 부름
```

### Intra-AS vs Inter-AS

| 구분 | 의미 | 예시 |
|---|---|---|
| Intra-AS | 같은 AS 내부 라우팅 | OSPF |
| Inter-AS | AS와 AS 사이 라우팅 | BGP |
| Gateway Router | 다른 AS와 연결된 경계 라우터 | 내부+외부 라우팅 수행 |

```text
[AS1 내부] --gateway-- [AS2 내부]
   OSPF                  OSPF
        
        AS 사이 정보 교환 = BGP
```

### OSPF

- Open Shortest Path First
- open = 공개 표준
- intra-AS 라우팅 프로토콜
- classic link-state 방식
- 각 라우터가 link-state advertisement를 flood
- TCP/UDP가 아니라 IP 위에서 직접 동작
- 각 라우터는 전체 AS topology를 알고 Dijkstra 수행
- 모든 OSPF 메시지는 인증 가능 → 보안 강화

### OSPF 핵심 흐름

```text
각 라우터가 자신의 link-state 광고
        ↓
AS 내부 전체에 flooding
        ↓
모든 라우터가 같은 topology 확보
        ↓
각자 Dijkstra 계산
        ↓
Forwarding Table 생성
```

### Hierarchical OSPF

- 2-level hierarchy
- local area + backbone area
- 영역 내부 정보는 영역 안에서만 자세히 유지
- backbone은 영역 간 연결 담당

| 라우터 | 역할 |
|---|---|
| Local Router | 자기 area 내부 라우팅 |
| Area Border Router | area 정보를 요약해 backbone에 전달 |
| Backbone Router | backbone 영역 라우팅 |
| Boundary Router | 다른 AS와 연결 |

```text
[Area 1] -- Area Border -- [Backbone] -- Area Border -- [Area 2]
                         |
                    Boundary Router
                         |
                       Other AS
```

---

## 6. BGP / Inter-AS Routing (5-50 ~ 5-63)

### BGP 핵심

- Border Gateway Protocol
- inter-domain routing의 사실상 표준
- “인터넷을 붙잡는 접착제”
- AS 사이에서 reachability 정보를 광고
- 성능보다 policy가 중요

### eBGP vs iBGP

| 구분 | 의미 | 역할 |
|---|---|---|
| eBGP | external BGP | 이웃 AS로부터 도달 가능 정보 획득 |
| iBGP | internal BGP | AS 내부 라우터들에게 정보 전파 |

```text
AS3 -- eBGP -- AS2 gateway
                 ↓ iBGP
              AS2 내부 전체
```

### BGP Session

- 두 BGP router(peer)가 TCP 연결로 메시지 교환
- semi-permanent TCP connection 사용
- BGP는 path vector protocol

### BGP route = prefix + attributes

| 요소 | 의미 |
|---|---|
| Prefix | 목적지 네트워크 주소 |
| AS-PATH | 광고가 지나온 AS 목록 |
| NEXT-HOP | 다음 AS로 나가기 위한 내부 라우터 |

```text
BGP 광고 예:
Prefix X에 가려면
AS2 → AS3 → X 경로 사용 가능

Route = X + {AS-PATH: AS2 AS3, NEXT-HOP: 2a}
```

### AS-PATH

- 목적지까지 거쳐야 하는 AS 목록
- loop 방지에 사용 가능
- 경로 선택 기준 중 하나

### NEXT-HOP

- 외부 목적지로 나가기 위한 AS 내부의 다음 gateway
- BGP 경로 정보와 intra-AS 라우팅이 연결되는 지점

### Policy-based routing

- AS 관리자가 정책에 따라 경로 선택
- 특정 AS를 통과하지 않게 설정 가능
- 고객/제공자 관계에 따라 광고 여부 결정

```text
성능상 최단 경로여도
정책상 허용되지 않으면 선택 X
```

### BGP 메시지 4종

| 메시지 | 역할 |
|---|---|
| OPEN | BGP 연결 열기 + peer 인증 |
| UPDATE | 새 경로 광고 / 기존 경로 철회 |
| KEEPALIVE | 연결 유지 |
| NOTIFICATION | 오류 보고 / 연결 종료 |

### Hot potato routing

- 뜨거운 감자를 오래 들고 있기 싫듯이
- 트래픽을 내 AS 안에서 오래 끌고 가지 않음
- 가장 가까운 gateway로 빨리 내보냄
- inter-domain 전체 최단이 아니라 intra-domain cost 최소가 우선

```text
목적지 X로 가는 gateway 후보: 2a, 2c
내 위치 2d 기준:
2d→2a 비용이 더 작다
        ↓
2a 선택
        ↓
AS 밖으로 빨리 내보냄
```

### BGP Route Selection 순서

```text
1. Local preference  ← 정책
2. Shortest AS-PATH
3. Closest NEXT-HOP  ← hot potato
4. 기타 기준
```

### 왜 Intra-AS와 Inter-AS를 나누는가

| 이유 | Intra-AS | Inter-AS |
|---|---|---|
| Policy | 중요도 낮음 | 매우 중요 |
| Scale | 하나의 AS 내부 | 인터넷 전체 규모 |
| Performance | 성능 중심 가능 | 정책이 성능보다 우선 |

---

## 7. SDN Control Plane (5-65 ~ 5-86)

### SDN 등장 배경

- 기존 라우터 = 하드웨어 + OS + 라우팅 프로토콜이 묶임
- vendor별 proprietary 구조 많음
- traffic engineering이 어려움
- link weight 조정만으로 원하는 경로 제어 한계

### SDN 핵심 정의

```text
SDN = Software Defined Networking
= Control Plane과 Data Plane 분리
= 중앙 컨트롤러가 네트워크 상태를 보고 flow table 설치
```

### SDN이 필요한 이유

- 중앙에서 네트워크 관리 쉬움
- 라우터 설정 오류 감소
- 트래픽 흐름을 더 유연하게 제어
- OpenFlow 같은 API로 switch 프로그래밍 가능
- control plane을 개방형으로 구현 가능

### SDN 4가지 특징

| 번호 | 특징 | 의미 |
|---|---|---|
| 1 | Flow-based forwarding | header field 기반 match+action |
| 2 | Control/Data 분리 | switch는 단순 forwarding |
| 3 | Control 기능 외부화 | controller가 control 담당 |
| 4 | Programmable app | routing, access control, load balance 앱 가능 |

### SDN 구조

```text
[Network Control Apps]
 routing / access control / load balance
              ↑
        Northbound API
              ↑
        [SDN Controller]
              ↓
        Southbound API
              ↓
[SDN Switches / Flow Tables]
```

### Northbound vs Southbound API

| 구분 | 방향 | 역할 |
|---|---|---|
| Northbound API | 앱 ↔ 컨트롤러 | 앱이 네트워크 상태/제어 기능 사용 |
| Southbound API | 컨트롤러 ↔ 스위치 | flow table 설치/수정/조회 |

### SDN Controller 역할

- network state 유지
- link / switch / host 정보 관리
- control app에 abstraction 제공
- switch와 통신
- 성능/확장성/장애 대응 위해 분산 시스템으로 구현 가능

### OpenFlow

- controller와 switch 사이에서 동작
- TCP로 메시지 교환
- 선택적으로 암호화 가능
- flow table 제어에 사용

### OpenFlow 메시지

| 방향 | 메시지 | 의미 |
|---|---|---|
| Controller → Switch | features | switch 기능 질의 |
| Controller → Switch | configure | 설정 조회/변경 |
| Controller → Switch | modify-state | flow entry 추가/삭제/수정 |
| Controller → Switch | packet-out | 특정 포트로 packet 전송 지시 |
| Switch → Controller | packet-in | packet을 controller로 전달 |
| Switch → Controller | flow-removed | flow entry 삭제 알림 |
| Switch → Controller | port-status | 포트 상태 변화 알림 |

### SDN 동작 예시

```text
1. switch에서 link failure 발생
2. switch가 OpenFlow port-status로 controller에 알림
3. controller가 network state 갱신
4. routing app이 변경 감지 후 새 경로 계산
5. controller가 새 flow table 계산
6. OpenFlow로 switch에 새 table 설치
```

### ODL / ONOS

| 컨트롤러 | 핵심 |
|---|---|
| OpenDaylight | Service Abstraction Layer, REST/NETCONF/SNMP 등 지원 |
| ONOS | intent 기반, 분산 core, reliability/scaling 강조 |

---

## 8. ICMP / Traceroute (5-87 ~ 5-89)

### ICMP

- Internet Control Message Protocol
- host와 router가 네트워크 수준 정보 전달
- 오류 보고 / ping / TTL 만료 등에 사용
- ICMP 메시지는 IP datagram 안에 담김

### 대표 ICMP 메시지

| Type/Code | 의미 | 사용 예 |
|---|---|---|
| 0/0 | echo reply | ping 응답 |
| 8/0 | echo request | ping 요청 |
| 3/0 | dest network unreachable | 네트워크 도달 불가 |
| 3/1 | dest host unreachable | 호스트 도달 불가 |
| 3/3 | dest port unreachable | 포트 도달 불가 |
| 11/0 | TTL expired | traceroute 중간 라우터 |
| 12/0 | bad IP header | IP 헤더 오류 |

### Traceroute 원리

```text
1. source가 TTL=1 UDP probe 전송
2. 1번째 router에서 TTL 만료
3. router가 ICMP TTL expired 반환
4. source가 RTT 기록
5. TTL=2, TTL=3 ... 반복
6. 목적지 도착 시 port unreachable 반환
7. source가 traceroute 종료
```

### 손필기식 암기

- ping = 살아있나 확인
- traceroute = 목적지까지 중간 라우터 추적
- TTL을 1씩 늘려가며 길을 드러냄

---

## 9. Network Management / SNMP / NETCONF / YANG (5-91 ~ 5-102)

### Network Management

- 네트워크 장비를 monitor / test / configure / analyze / control
- 대상: router, switch, server, interface 등
- 목적: 성능, QoS, 장애 대응, 설정 관리

### 구성 요소

| 요소 | 역할 |
|---|---|
| Managing Server | 관리자가 사용하는 제어 서버 |
| Managed Device | 관리 대상 장비 |
| Agent | 장비 내부에서 상태 제공/명령 수행 |
| Management Protocol | 서버와 장비 사이 통신 규칙 |
| Data | 설정값, 운영 상태, 통계 |

```text
[Managing Server]
        ↓ query / config
[Agent in Managed Device]
        ↓
[Device state / config / statistics]
```

### 관리 방식 3가지

| 방식 | 핵심 | 한계/특징 |
|---|---|---|
| CLI | 장비에 직접 명령 | 장비별 수동 관리 느낌 |
| SNMP/MIB | 장비 데이터 조회/설정 | 상태 모니터링 중심 |
| NETCONF/YANG | 구조화된 설정 관리 | 여러 장비 설정에 적합 |

### SNMP

- Simple Network Management Protocol
- managing server가 agent에게 데이터 요청/설정
- agent가 예외 상황을 trap으로 알림

```text
Request/Response:
Manager → Agent: 값 알려줘 / 값 설정해
Agent → Manager: 응답

Trap:
Agent → Manager: 이상 상황 발생!
```

### SNMP 메시지

| 메시지 | 방향 | 의미 |
|---|---|---|
| GetRequest | manager → agent | 특정 데이터 요청 |
| GetNextRequest | manager → agent | 다음 데이터 요청 |
| GetBulkRequest | manager → agent | 여러 데이터 묶음 요청 |
| SetRequest | manager → agent | MIB 값 설정 |
| Response | agent → manager | 요청 응답 |
| Trap | agent → manager | 예외 이벤트 알림 |

### MIB

- Management Information Base
- managed device의 운영/설정 데이터 모음
- object ID로 각 값을 식별
- 예: UDPInDatagrams, UDPNoPorts 등

### NETCONF

- 네트워크 장비 설정을 능동적으로 관리하기 위한 프로토콜
- retrieve / set / modify / activate configuration 가능
- RPC 방식 사용
- XML로 메시지 인코딩
- secure reliable transport 사용 가능

### NETCONF 흐름

```text
1. <hello>로 session 시작 + capability 교환
2. <rpc> 요청
3. <rpc-reply> 응답
4. 필요하면 <notification>
5. <close-session>으로 종료
```

### 대표 NETCONF Operation

| Operation | 의미 |
|---|---|
| <get-config> | 설정 정보 조회 |
| <get> | 설정 + 운영 상태 조회 |
| <edit-config> | 설정 변경 |
| <lock>/<unlock> | 설정 저장소 잠금/해제 |
| <create-subscription> | 이벤트 알림 구독 |
| <notification> | 알림 전달 |

### YANG

- NETCONF 데이터 모델링 언어
- 설정 데이터의 구조/문법/의미 정의
- 유효한 설정인지 제약 조건 표현 가능
- YANG → XML 구조 생성 가능

```text
YANG = 설정 데이터의 설계도
NETCONF = 그 설계도에 맞춰 장비와 주고받는 통신 방식
```

---

## 10. 마지막 비교 정리

### Control Plane 구조 비교

| 항목 | Per-router | SDN |
|---|---|---|
| 계산 위치 | 각 라우터 | 중앙 controller |
| 구조 | 분산 | 논리적 중앙집중 |
| 장점 | 전통적, 독립 동작 | 관리 쉬움, 프로그래밍 가능 |
| 단점 | 경로 제어 어려움 | controller 신뢰성 중요 |

### Routing Algorithm 비교

| 항목 | Link State | Distance Vector |
|---|---|---|
| 정보 | 전체 topology | 이웃 DV |
| 대표 알고리즘 | Dijkstra | Bellman-Ford |
| 계산 방식 | 전체 지도 기반 | 이웃 정보 기반 |
| 문제 | Oscillation | Count-to-infinity |
| 구현 예 | OSPF | RIP |

### Intra-AS vs Inter-AS

| 항목 | Intra-AS | Inter-AS |
|---|---|---|
| 범위 | AS 내부 | AS 사이 |
| 대표 | OSPF | BGP |
| 관심 | 성능 | 정책 |
| 관리자 | 하나의 조직 | 여러 조직 |

### OSPF vs BGP

| 항목 | OSPF | BGP |
|---|---|---|
| 위치 | AS 내부 | AS 사이 |
| 방식 | Link-State | Path Vector |
| 기준 | cost 기반 최단 경로 | policy + AS-PATH + NEXT-HOP |
| 특징 | Dijkstra 사용 | 인터넷 inter-domain 표준 |

### SNMP vs NETCONF/YANG

| 항목 | SNMP/MIB | NETCONF/YANG |
|---|---|---|
| 중심 | 상태 조회/간단 설정 | 구조화된 설정 관리 |
| 데이터 모델 | MIB/SMI | YANG |
| 통신 | Get/Set/Trap | RPC/XML |
| 느낌 | 장비 모니터링 중심 | 네트워크 설정 자동화 중심 |

---

## 11. 최종 체크리스트

- [ ] Data Plane과 Control Plane 차이를 설명할 수 있다.
- [ ] Forwarding과 Routing 차이를 설명할 수 있다.
- [ ] Per-router와 SDN 구조를 구분할 수 있다.
- [ ] Link State와 Distance Vector 차이를 말할 수 있다.
- [ ] Dijkstra 표에서 N', D(v), p(v)를 읽을 수 있다.
- [ ] Bellman-Ford 식의 의미를 설명할 수 있다.
- [ ] Good news / Bad news 차이를 설명할 수 있다.
- [ ] Count-to-infinity가 왜 생기는지 설명할 수 있다.
- [ ] AS, intra-AS, inter-AS를 구분할 수 있다.
- [ ] OSPF가 왜 Link-State 기반인지 설명할 수 있다.
- [ ] BGP의 AS-PATH, NEXT-HOP 의미를 설명할 수 있다.
- [ ] Hot potato routing을 예시로 설명할 수 있다.
- [ ] SDN에서 northbound/southbound API를 구분할 수 있다.
- [ ] ICMP와 traceroute의 관계를 설명할 수 있다.
- [ ] SNMP/MIB와 NETCONF/YANG 차이를 설명할 수 있다.

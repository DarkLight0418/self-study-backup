# Chapter 5 Network Layer: Control Plane — 필기용 노트

## 0. Chapter 5 전체 흐름

Chapter 5는 네트워크 계층의 **Control Plane**을 다룬다.  
핵심 질문은 “패킷을 어디로 보낼지 결정하는 forwarding table은 누가, 어떻게 만드는가?”이다.

```text
Network Layer
├─ Data Plane      : forwarding 담당
│                  : 들어온 패킷을 어느 출력 포트로 보낼지 처리
│
└─ Control Plane   : routing 담당
                   : 목적지까지의 경로를 계산하고 forwarding table을 만듦
```

이 장의 흐름은 다음 순서로 잡으면 된다.

```text
Control Plane 구조
→ Routing Algorithm
→ Link State / Distance Vector
→ OSPF / BGP
→ SDN Control Plane
→ ICMP
→ Network Management: SNMP, NETCONF/YANG
```

정리하면, Chapter 5는 “길을 계산하는 방식”에서 시작해서 “인터넷 규모에서 그 길을 관리하는 방식”까지 확장되는 장이다.

---

## 1. Data Plane과 Control Plane

네트워크 계층에는 크게 두 기능이 있다.

| 구분          | 핵심 역할  | 쉽게 말하면                         |
| ------------- | ---------- | ----------------------------------- |
| Data Plane    | forwarding | 이미 정해진 표를 보고 패킷을 내보냄 |
| Control Plane | routing    | 그 표가 어떻게 만들어질지 결정함    |

**Forwarding**은 라우터 내부에서 매우 빠르게 일어나는 동작이다.  
패킷이 들어오면 헤더 값을 보고 forwarding table을 조회한 뒤 적절한 출력 포트로 보낸다.

**Routing**은 목적지까지 어떤 경로를 사용할지 계산하는 과정이다.  
라우팅 알고리즘이나 라우팅 프로토콜이 여러 라우터의 정보를 바탕으로 forwarding table을 만든다.

```text
패킷 도착
→ 헤더 확인
→ forwarding table 조회
→ 출력 포트 선택
→ 다음 라우터로 전달
```

핵심 구분은 다음 한 문장으로 외우면 된다.

> Data Plane은 만들어진 길을 따라 보내는 역할이고, Control Plane은 그 길을 만드는 역할이다.

---

## 2. Control Plane을 구성하는 두 방식

Control Plane을 구성하는 대표 방식은 두 가지이다.

| 구분             | Per-router Control         | SDN Control                |
| ---------------- | -------------------------- | -------------------------- |
| 제어 위치        | 각 라우터 내부             | 원격 컨트롤러              |
| 계산 방식        | 라우터들이 분산적으로 계산 | 컨트롤러가 전체적으로 계산 |
| forwarding table | 각 라우터가 직접 만듦      | 컨트롤러가 계산해서 설치   |
| 감각             | 각자 알아서 길 찾기        | 관제센터가 길 지시         |

### 2-1. Per-router Control Plane

전통적인 라우팅 방식이다.  
각 라우터 안에 라우팅 알고리즘이 들어 있고, 라우터들이 서로 정보를 주고받으면서 각자의 forwarding table을 만든다.

```text
[Router A] ↔ [Router B] ↔ [Router C]
   │             │             │
   └ 각 라우터가 자기 라우팅 알고리즘으로 forwarding table 계산
```

이 방식에서는 네트워크 전체를 하나의 중앙 장치가 직접 제어하지 않는다.  
대신 각 라우터가 주변 정보나 전체 링크 상태 정보를 이용해 자기 표를 만든다.

### 2-2. SDN Control Plane

SDN은 **Software-Defined Networking**의 약자이다.  
라우터나 스위치가 모든 제어 기능을 직접 수행하지 않고, 원격 컨트롤러가 forwarding table을 계산해서 장비에 설치한다.

```text
              [SDN Controller]
             /       |       \
            v        v        v
       [Switch1] [Switch2] [Switch3]
```

SDN의 핵심은 **Control Plane과 Data Plane의 분리**이다.  
스위치는 빠르게 forwarding하고, 컨트롤러는 전체 네트워크 상태를 보고 제어 규칙을 계산한다.

정리하면, 전통적 방식은 “각 라우터가 스스로 생각하는 구조”이고, SDN은 “컨트롤러가 계산하고 스위치가 실행하는 구조”이다.

---

## 3. Routing Protocol의 목표

라우팅 프로토콜의 목표는 출발지에서 목적지까지 가는 **좋은 path**를 찾는 것이다.

여기서 path는 패킷이 지나가는 라우터들의 순서이다.

```text
Source Host → R1 → R2 → R3 → Destination Host
```

좋은 path의 기준은 상황에 따라 다르다.

| 기준            | 의미                  |
| --------------- | --------------------- |
| least cost      | 비용이 가장 낮은 경로 |
| fastest         | 지연 시간이 짧은 경로 |
| least congested | 혼잡이 적은 경로      |

교안에서는 주로 cost를 기준으로 설명한다.  
즉, 라우팅 문제는 “여러 경로 중 총 비용이 가장 낮은 경로를 찾는 문제”로 볼 수 있다.

---

## 4. Graph Abstraction과 Link Cost

라우팅 알고리즘은 네트워크를 그래프로 추상화한다.

```text
G = (N, E)
N = router들의 집합
E = router 사이 link들의 집합
c(x,y) = x와 y 사이 직접 링크 비용
```

직접 연결된 라우터 사이에는 link cost가 존재한다.  
직접 연결되지 않은 경우에는 비용을 ∞로 둔다.

```text
직접 연결됨       : c(x,y) = 실제 비용
직접 연결 안 됨   : c(x,y) = ∞
```

Link cost는 운영자가 정할 수 있다.  
모든 링크 비용을 1로 둘 수도 있고, bandwidth가 클수록 비용을 낮게 잡을 수도 있고, 혼잡이 심할수록 비용을 높게 잡을 수도 있다.

따라서 cost는 단순한 돈의 의미가 아니라 “그 링크를 사용하는 부담”이라고 이해하면 된다.

---

## 5. Routing Algorithm 분류

라우팅 알고리즘은 정보를 얼마나 알고 계산하는지에 따라 나눌 수 있다.

| 기준          | Link State                              | Distance Vector                |
| ------------- | --------------------------------------- | ------------------------------ |
| 정보 범위     | 전체 네트워크 topology와 link cost를 앎 | 직접 연결된 이웃 정보에서 시작 |
| 계산 방식     | 전체 지도를 보고 계산                   | 이웃이 알려준 거리 정보를 이용 |
| 대표 알고리즘 | Dijkstra                                | Bellman-Ford                   |
| 감각          | 전체 지도를 들고 계산                   | 이웃에게 물어보며 계산         |

또한 경로가 얼마나 자주 바뀌는지에 따라 static routing과 dynamic routing으로도 나눈다.

| 구분            | 의미                                          |
| --------------- | --------------------------------------------- |
| Static Routing  | 경로가 천천히 바뀌거나 수동 설정 중심         |
| Dynamic Routing | 링크 비용 변화나 장애에 따라 경로가 자동 갱신 |

한 문장으로 정리하면 다음과 같다.

> Link State는 전체 지도를 알고 계산하고, Distance Vector는 이웃이 알려준 정보로 조금씩 계산한다.

---

## 6. Link State Routing과 Dijkstra Algorithm

Link State 방식에서는 모든 라우터가 전체 네트워크 topology와 link cost 정보를 알고 있다고 가정한다.  
이 정보는 link state broadcast를 통해 공유된다.

각 라우터는 같은 전체 지도를 가지고 있지만, 자기 자신을 출발점으로 두고 forwarding table을 계산한다.

### 6-1. Dijkstra 알고리즘의 목적

Dijkstra 알고리즘은 한 출발지에서 모든 목적지까지의 최소 비용 경로를 구한다.

```text
출발지 u 기준
u → v 최소 비용
u → w 최소 비용
u → x 최소 비용
u → y 최소 비용
u → z 최소 비용
```

계산이 끝나면 출발지 라우터의 forwarding table을 만들 수 있다.

### 6-2. Dijkstra에서 쓰는 기호

| 기호   | 의미                                              |
| ------ | ------------------------------------------------- |
| c(x,y) | x와 y 사이 직접 링크 비용, 직접 연결 아니면 ∞     |
| D(v)   | 출발지에서 v까지 가는 현재까지의 최소 비용 추정값 |
| p(v)   | v로 가는 최소 경로에서 v 바로 이전 노드           |
| N'     | 최소 비용이 확정된 노드들의 집합                  |

### 6-3. Dijkstra 계산 흐름

Dijkstra는 “가장 가까운 노드를 하나씩 확정하고, 그 노드를 통해 다른 노드 비용을 갱신”한다.

```text
1. 출발지 u를 N'에 넣는다.
2. u와 직접 연결된 노드의 D 값을 초기화한다.
3. N'에 없는 노드 중 D 값이 가장 작은 노드 w를 고른다.
4. w를 N'에 추가한다.
5. w와 인접한 노드 v에 대해 비용을 갱신한다.
6. 모든 노드가 N'에 들어갈 때까지 반복한다.
```

핵심 갱신식은 다음과 같다.

```text
D(v) = min(기존 D(v), D(w) + c(w,v))
```

뜻은 간단하다.

```text
v로 가는 기존 경로가 더 싼가?
아니면 새로 확정한 w를 거쳐 v로 가는 경로가 더 싼가?
둘 중 더 싼 값을 D(v)로 둔다.
```

직접 연결되지 않은 노드는 ∞로 둔다.  
∞는 “지금 알고 있는 직접 경로가 없다”는 뜻이다.

### 6-4. Dijkstra 표를 읽는 방법

Dijkstra 표는 보통 다음 열을 가진다.

```text
Step | N' | D(v),p(v) | D(w),p(w) | D(x),p(x) | ...
```

여기서 N'는 “최소 비용 경로가 확정된 노드 집합”이다.  
각 D값 옆의 p값은 predecessor, 즉 직전 노드이다.

경로를 복원할 때는 predecessor를 거꾸로 따라가면 된다.

```text
예: p(z)=y, p(y)=x, p(x)=u
그러면 u에서 z까지 경로는 u → x → y → z
```

### 6-5. Dijkstra의 복잡도와 주의점

기본 구현에서는 매 단계마다 아직 확정되지 않은 노드 중 최소 D값을 찾아야 하므로 O(n²) 정도가 된다.  
더 효율적인 자료구조를 쓰면 O(n log n) 수준으로 개선할 수 있다.

메시지 관점에서는 각 라우터가 자신의 link state 정보를 다른 라우터들에게 알려야 한다.  
그래서 전체 네트워크에 link state 정보를 퍼뜨리는 비용도 중요하다.

Dijkstra의 주의점은 **oscillation**이다.  
만약 link cost가 트래픽 양에 따라 변하면, 경로가 바뀌고 트래픽이 이동하면서 다시 cost가 변하고, 그 결과 경로가 계속 흔들릴 수 있다.

```text
트래픽 몰림
→ 해당 link cost 증가
→ 라우팅 알고리즘이 다른 경로 선택
→ 트래픽이 다른 경로로 이동
→ 그 경로의 cost 증가
→ 다시 경로 변경
```

이처럼 경로가 안정되지 않고 반복적으로 바뀌는 현상을 oscillation이라고 한다.

---

## 7. Distance Vector Routing과 Bellman-Ford

Distance Vector 방식에서는 각 라우터가 처음부터 전체 네트워크 지도를 알지 못한다.  
처음에는 직접 연결된 이웃까지의 비용만 알고, 이웃들이 보내는 distance vector를 받아 자신의 정보를 갱신한다.

### 7-1. Distance Vector의 핵심 감각

Distance Vector는 “목적지 y까지 가려면 어떤 이웃을 거쳐 가는 것이 가장 싼가?”를 반복해서 계산한다.

```text
내가 y로 직접 가는 길을 모를 수도 있음
→ 이웃 v는 y까지 비용을 알고 있을 수 있음
→ 그럼 나는 x→v 비용 + v→y 비용을 계산
→ 여러 이웃 중 최소값 선택
```

### 7-2. Bellman-Ford 식

Distance Vector의 핵심 식은 다음과 같다.

```text
D_x(y) = min_v { c(x,v) + D_v(y) }
```

각 기호의 의미는 다음과 같다.

| 기호   | 의미                                  |
| ------ | ------------------------------------- |
| D_x(y) | x에서 y까지 가는 최소 비용 추정값     |
| c(x,v) | x에서 이웃 v까지 가는 직접 비용       |
| D_v(y) | 이웃 v가 알고 있는 v에서 y까지의 비용 |
| min_v  | 모든 이웃 v 중 최소값 선택            |

즉, 내 이웃들 중 누가 목적지까지 가장 싸게 데려다줄 수 있는지 비교하는 식이다.

### 7-3. Distance Vector의 동작 흐름

```text
1. 각 노드는 처음에 직접 연결된 이웃 비용만 안다.
2. 각 노드는 자기 distance vector를 이웃에게 보낸다.
3. 이웃의 distance vector를 받은 노드는 Bellman-Ford 식으로 갱신한다.
4. 자신의 distance vector가 바뀌면 다시 이웃에게 알린다.
5. 더 이상 바뀌지 않으면 멈춘다.
```

Distance Vector는 **iterative**, **asynchronous**, **distributed**, **self-stopping** 특성을 가진다.

| 특성          | 의미                                       |
| ------------- | ------------------------------------------ |
| iterative     | 반복 계산을 통해 점점 정확해짐             |
| asynchronous  | 모든 라우터가 동시에 맞춰 움직일 필요 없음 |
| distributed   | 중앙 계산기가 아니라 각 노드가 나눠 계산   |
| self-stopping | 바뀐 정보가 없으면 더 이상 알리지 않음     |

### 7-4. 정보 확산 감각

Distance Vector에서는 정보가 한 번에 전체로 퍼지지 않는다.  
이웃에서 이웃으로 조금씩 퍼진다.

```text
t=0 : 어떤 정보가 해당 노드에만 있음
t=1 : 1-hop 이웃까지 영향
t=2 : 2-hop 이웃까지 영향
t=3 : 3-hop 이웃까지 영향
```

따라서 전체 네트워크가 안정된 경로를 알기까지 시간이 걸릴 수 있다.

### 7-5. Good News Travels Fast

링크 비용이 낮아지는 좋은 변화는 비교적 빨리 퍼진다.

```text
어떤 링크 비용 감소
→ 해당 노드가 더 좋은 경로 발견
→ 이웃에게 알림
→ 이웃도 더 좋은 경로 발견
→ 빠르게 전파
```

좋은 경로는 “내가 더 싸게 갈 수 있다”는 정보이므로 주변 노드들이 쉽게 받아들인다.

### 7-6. Bad News Travels Slow와 Count-to-Infinity

문제는 링크 비용이 크게 증가하거나 경로가 끊기는 경우이다.  
나쁜 소식은 천천히 퍼질 수 있고, 이때 **count-to-infinity** 문제가 발생한다.

예를 들어 y가 x로 가는 직접 링크 비용이 4에서 60으로 증가했다고 하자.  
그런데 z가 예전 정보 때문에 “나는 x까지 5로 갈 수 있어”라고 말하면, y는 z를 거쳐 x로 가면 6이라고 착각할 수 있다.

```text
y: x까지 직접 비용이 60이 되었네.
z: 나는 x까지 5로 갈 수 있어.
y: 그럼 나는 z를 거치면 1+5=6이네?
z: y가 x까지 6이라고 하네. 그럼 나는 y를 거치면 7이네?
y: z가 x까지 7이라고 하네. 그럼 나는 z를 거치면 8이네?
...
```

이렇게 서로가 서로의 잘못된 정보를 믿으면서 비용이 6, 7, 8, 9처럼 천천히 증가한다.  
이 현상을 count-to-infinity라고 한다.

핵심은 다음과 같다.

> Distance Vector는 이웃의 말을 믿고 계산하기 때문에, 잘못된 경로 정보가 순환하면 수렴이 느려질 수 있다.

---

## 8. Link State와 Distance Vector 비교

| 구분          | Link State                                  | Distance Vector                       |
| ------------- | ------------------------------------------- | ------------------------------------- |
| 정보          | 전체 topology와 link cost                   | 이웃의 distance vector                |
| 대표 알고리즘 | Dijkstra                                    | Bellman-Ford                          |
| 계산 위치     | 각 라우터가 전체 지도 기반 계산             | 각 라우터가 이웃 정보 기반 계산       |
| 메시지        | link state 정보를 전체에 flood              | 이웃끼리 distance vector 교환         |
| 장점          | 전체 구조를 보고 계산하므로 명확함          | 구조가 단순하고 분산적                |
| 단점          | link state broadcast 부담, oscillation 가능 | routing loop, count-to-infinity 가능  |
| 오류 영향     | 잘못된 link cost 광고 가능                  | 잘못된 path cost가 주변으로 전파 가능 |

LS에서 한 라우터가 잘못된 link cost를 광고할 수 있다.  
하지만 각 라우터는 자기 forwarding table을 자기 알고리즘으로 계산한다.

DV에서는 한 라우터가 “나는 모든 곳까지 싸게 갈 수 있다”고 잘못 광고하면, 다른 라우터들이 그 정보를 믿고 트래픽을 보낼 수 있다.  
이 경우 트래픽이 잘못된 라우터로 빨려 들어가는 **black-holing** 문제가 생길 수 있다.

---

## 9. 실제 인터넷에서 라우팅을 확장하는 이유

지금까지의 라우팅 설명은 네트워크가 평평하다고 가정한다.  
즉, 모든 라우터가 같은 수준에서 하나의 네트워크처럼 동작한다고 보는 이상화된 모델이다.

하지만 실제 인터넷은 그렇지 않다.

```text
문제 1: 규모가 너무 큼
→ 수십억 개 목적지를 하나의 routing table에 모두 넣기 어려움
→ 모든 라우팅 정보를 계속 교환하면 링크가 감당하기 어려움

문제 2: 관리 주체가 다름
→ 인터넷은 network of networks
→ 각 ISP, 기업, 기관이 자기 네트워크 정책을 따로 가짐
```

그래서 인터넷은 **Autonomous System**, 줄여서 **AS** 단위로 나눈다.

AS는 하나의 관리 주체가 운영하는 네트워크 묶음이다.  
예를 들면 하나의 ISP 네트워크, 대학 네트워크, 기업 네트워크가 AS가 될 수 있다.

---

## 10. Intra-AS Routing과 Inter-AS Routing

AS를 기준으로 라우팅은 두 종류로 나뉜다.

| 구분             | 의미                       | 대표 예 |
| ---------------- | -------------------------- | ------- |
| Intra-AS Routing | 같은 AS 내부에서의 라우팅  | OSPF    |
| Inter-AS Routing | 서로 다른 AS 사이의 라우팅 | BGP     |

같은 AS 안에서는 하나의 관리자가 운영하므로 성능 중심으로 경로를 정하기 쉽다.  
반면 AS 사이에서는 성능보다 정책이 중요하다.

```text
AS 내부 목적지
→ intra-AS routing만으로 forwarding table 결정

AS 외부 목적지
→ inter-AS routing으로 어느 gateway로 나갈지 결정
→ intra-AS routing으로 그 gateway까지 가는 내부 경로 결정
```

Gateway router는 자기 AS의 가장자리에 있으며 다른 AS와 연결된 라우터이다.  
외부 목적지로 가는 패킷은 보통 AS 내부에서 gateway router까지 이동한 뒤 다른 AS로 넘어간다.

---

## 11. OSPF: Intra-AS Link State Routing Protocol

OSPF는 **Open Shortest Path First**의 약자이다.  
대표적인 intra-AS 라우팅 프로토콜이며 link-state 방식을 사용한다.

OSPF의 핵심은 다음과 같다.

```text
OSPF = 공개 표준 + Link State + Dijkstra 기반 + AS 내부 라우팅
```

OSPF에서 각 라우터는 자신의 link state advertisement를 AS 내부의 다른 라우터들에게 flood한다.  
각 라우터는 전체 AS 내부 topology를 알게 되고, Dijkstra 알고리즘을 사용해 forwarding table을 계산한다.

OSPF 메시지는 TCP나 UDP가 아니라 IP 위에서 직접 동작한다.  
또한 OSPF 메시지는 인증될 수 있어 악의적인 라우팅 정보 삽입을 막는 데 도움을 준다.

### 11-1. 계층적 OSPF

큰 AS에서는 모든 라우터에게 모든 link state 정보를 flood하면 부담이 커진다.  
그래서 OSPF는 계층 구조를 사용할 수 있다.

```text
Backbone Area
├─ Area 1
├─ Area 2
└─ Area 3
```

계층적 OSPF에는 여러 종류의 라우터가 등장한다.

| 라우터                | 역할                                                     |
| --------------------- | -------------------------------------------------------- |
| Local/Internal Router | 자기 area 내부에서 link state flood, area 내부 경로 계산 |
| Area Border Router    | 자기 area 정보를 요약해서 backbone에 전달                |
| Backbone Router       | backbone 영역에서 OSPF 수행                              |
| Boundary Router       | 다른 AS와 연결                                           |

계층적 OSPF의 핵심은 “모든 세부 정보를 전체에 뿌리지 않고, area 안에서는 자세히 알고 area 밖은 방향 중심으로 아는 것”이다.

---

## 12. BGP: Inter-AS Routing Protocol

BGP는 **Border Gateway Protocol**의 약자이다.  
인터넷에서 AS와 AS를 연결하는 대표적인 inter-AS 라우팅 프로토콜이다.

BGP는 흔히 “인터넷을 묶는 접착제”라고 표현된다.  
각 AS가 자신이 어떤 네트워크에 도달할 수 있는지를 다른 AS에게 알려주기 때문이다.

```text
BGP의 핵심 문장
“나는 여기 있고, 이 목적지들까지 갈 수 있으며, 그 경로는 이렇다.”
```

BGP에는 eBGP와 iBGP가 있다.

| 구분 | 의미                                                             |
| ---- | ---------------------------------------------------------------- |
| eBGP | 서로 다른 AS의 gateway router끼리 reachability 정보를 교환       |
| iBGP | 외부에서 배운 reachability 정보를 같은 AS 내부 라우터들에게 전파 |

Gateway router는 eBGP와 iBGP를 모두 수행할 수 있다.

---

## 13. BGP Session과 Path Vector

BGP에서는 두 BGP 라우터를 peer라고 부른다.  
BGP peer들은 TCP 연결을 맺고 BGP message를 교환한다.

BGP는 단순한 distance vector가 아니라 **path vector** 프로토콜이다.  
목적지까지의 비용만 말하는 것이 아니라, 어떤 AS들을 거쳐 가는지도 함께 알린다.

```text
AS3가 X까지 갈 수 있다고 AS2에 광고
→ 광고 내용: AS3, X
→ 의미: AS3는 X 방향으로 패킷을 전달해줄 수 있음
```

BGP가 광고하는 route는 다음 두 부분으로 구성된다.

```text
BGP route = prefix + attributes
```

| 요소     | 의미                                                  |
| -------- | ----------------------------------------------------- |
| prefix   | 광고되는 목적지 네트워크                              |
| AS-PATH  | 해당 prefix 광고가 거쳐 온 AS들의 목록                |
| NEXT-HOP | 다음 AS로 나가기 위해 내부에서 가야 할 gateway router |

AS-PATH는 loop를 막고 정책 판단에도 사용된다.  
NEXT-HOP은 AS 내부 라우터가 외부 목적지로 가기 위해 어떤 gateway를 향해야 하는지 알려준다.

---

## 14. BGP Path Advertisement 흐름

BGP path advertisement는 “도달 가능성 정보가 AS에서 AS로 전파되는 과정”이다.

예를 들어 X라는 네트워크가 AS3에 붙어 있다고 하자.

```text
1. AS3 gateway가 AS2 gateway에게 “AS3, X”를 광고한다.
2. AS2는 정책상 이 경로를 받아들일지 결정한다.
3. AS2가 받아들이면 iBGP로 AS2 내부 라우터들에게 전파한다.
4. AS2가 AS1에게 광고할 때는 “AS2, AS3, X”처럼 자기 AS를 앞에 붙인다.
5. AS1도 여러 경로 중 정책에 맞는 경로를 선택한다.
```

BGP에서 중요한 점은 “최단 거리”만 보고 선택하지 않는다는 것이다.  
AS마다 정책이 있기 때문에 어떤 경로는 받아들이지 않거나, 어떤 경로는 다른 AS에게 광고하지 않을 수 있다.

---

## 15. BGP 메시지

BGP 메시지는 TCP 연결 위에서 교환된다.

| 메시지       | 역할                                    |
| ------------ | --------------------------------------- |
| OPEN         | BGP peer와 연결을 열고 인증             |
| UPDATE       | 새 경로를 광고하거나 기존 경로를 철회   |
| KEEPALIVE    | UPDATE가 없어도 연결이 살아 있음을 확인 |
| NOTIFICATION | 오류 보고 또는 연결 종료                |

BGP에서 가장 핵심적인 메시지는 UPDATE이다.  
UPDATE가 새로운 path advertisement나 withdraw 정보를 전달하기 때문이다.

---

## 16. Intra-AS와 Inter-AS 라우팅이 다른 이유

Intra-AS와 Inter-AS 라우팅이 다른 이유는 세 가지이다.

| 기준 | Intra-AS                          | Inter-AS                       |
| ---- | --------------------------------- | ------------------------------ |
| 정책 | 한 관리자 내부라 정책 문제가 작음 | AS마다 정책과 이해관계가 큼    |
| 규모 | AS 내부 범위                      | 인터넷 전체 규모               |
| 성능 | 성능 중심 최적화 가능             | 성능보다 정책이 우선될 수 있음 |

AS 내부에서는 “빠른 경로”가 중요하다.  
AS 사이에서는 “누구의 트래픽을 내 네트워크에 통과시킬 것인가”, “어떤 AS를 거치지 않을 것인가” 같은 정책이 중요하다.

따라서 BGP는 단순 최단 경로 프로토콜이 아니라 정책 기반 라우팅 프로토콜이라고 이해해야 한다.

---

## 17. Hot Potato Routing

Hot potato routing은 BGP 경로 선택에서 나오는 개념이다.  
뜨거운 감자를 오래 들고 있기 싫어서 빨리 밖으로 던지는 것처럼, 자기 AS 내부에서 가장 가까운 gateway를 통해 트래픽을 내보내는 방식이다.

```text
목적지 X로 가는 외부 경로가 여러 개 있음
→ 내 AS 내부 비용이 가장 작은 gateway 선택
→ AS 밖의 전체 경로 비용은 깊게 고려하지 않음
```

예를 들어 AS2의 어떤 라우터가 X로 가기 위해 2a gateway 또는 2c gateway를 쓸 수 있다고 하자.  
Hot potato routing에서는 AS 전체 관점에서 더 좋은 외부 경로인지보다, 현재 라우터에서 더 가까운 gateway를 고를 수 있다.

BGP route selection의 대표 순서는 다음처럼 이해하면 된다.

```text
1. Local preference: 정책상 선호하는 경로인가?
2. AS-PATH 길이: 거치는 AS 수가 짧은가?
3. Closest NEXT-HOP: 내부 비용이 가장 낮은 gateway인가?
4. 기타 기준
```

핵심은 “BGP는 성능보다 정책을 먼저 본다”는 점이다.

---

## 18. BGP Policy 예시

ISP는 보통 자기 고객과 관련된 트래픽은 전달해도, 다른 ISP들 사이의 단순 통과 트래픽은 원하지 않을 수 있다.  
왜냐하면 수익이 없는 transit traffic을 자기 네트워크가 대신 운반하게 되기 때문이다.

```text
A, B, C = provider network
w, x, y = customer network

B는 C가 A의 고객 w로 가는 트래픽을 자기 네트워크를 통해 보내는 것을 원하지 않을 수 있음
→ 그래서 B는 특정 경로를 C에게 광고하지 않을 수 있음
```

BGP에서는 경로를 “알리는 것” 자체가 정책 수단이다.  
광고하지 않으면 상대 AS는 그 경로를 선택할 수 없다.

Dual-homed customer도 정책을 가질 수 있다.  
예를 들어 x가 두 provider에 연결되어 있어도, x가 B에서 C로 가는 통과 경로가 되고 싶지 않다면 특정 route를 광고하지 않으면 된다.

---

## 19. SDN이 필요한 이유

전통적 라우팅에서는 운영자가 원하는 세밀한 트래픽 제어를 하기 어렵다.  
왜냐하면 주된 제어 수단이 link weight 조정이기 때문이다.

예를 들어 다음 요구를 생각해보자.

```text
요구 1: u→z 트래픽을 특정 경로로 보내고 싶다.
요구 2: u→z 트래픽을 두 경로로 나누고 싶다.
요구 3: 같은 목적지라도 빨간 트래픽과 파란 트래픽을 다르게 보내고 싶다.
```

전통적인 destination-based forwarding과 LS/DV routing만으로는 이런 요구를 직접 표현하기 어렵다.  
링크 비용을 억지로 조정하거나 새로운 알고리즘이 필요할 수 있다.

SDN은 flow-based forwarding을 사용하므로 더 세밀한 제어가 가능하다.

```text
전통적 라우팅: 목적지 중심
SDN: flow 중심, 조건 기반 forwarding 가능
```

---

## 20. SDN의 핵심 구성

SDN의 핵심 특징은 네 가지로 정리할 수 있다.

```text
1. Generalized flow-based forwarding
2. Control plane과 data plane 분리
3. Control 기능을 data-plane switch 밖으로 이동
4. Programmable control applications
```

SDN 구조는 다음처럼 볼 수 있다.

```text
[Network Control Applications]
   예: routing, access control, load balancing
              │
              │ Northbound API
              v
[SDN Controller = Network OS]
              │
              │ Southbound API
              v
[SDN-controlled Switches]
```

### 20-1. Data-plane Switch

SDN switch는 빠르고 단순한 forwarding 장비이다.  
flow table은 컨트롤러의 감독 아래 계산되고 설치된다.

### 20-2. SDN Controller

SDN controller는 network operating system처럼 동작한다.  
네트워크 상태를 유지하고, 위쪽 control application과 아래쪽 switch 사이를 연결한다.

| API            | 연결 대상                        | 역할                                  |
| -------------- | -------------------------------- | ------------------------------------- |
| Northbound API | control application ↔ controller | 앱이 네트워크 상태와 제어 기능을 사용 |
| Southbound API | controller ↔ switch              | 스위치 상태 수집, flow rule 설치      |

### 20-3. Network Control Application

Control application은 SDN의 실제 “두뇌” 역할을 한다.  
예를 들어 routing app, firewall app, load balancing app이 컨트롤러 API를 사용해 네트워크 정책을 구현한다.

---

## 21. OpenFlow

OpenFlow는 SDN controller와 switch 사이에서 동작하는 대표 프로토콜이다.  
TCP 연결을 통해 메시지를 주고받으며, 선택적으로 암호화를 사용할 수 있다.

OpenFlow 메시지는 크게 세 종류로 나뉜다.

| 종류                 | 방향                | 의미                         |
| -------------------- | ------------------- | ---------------------------- |
| Controller-to-switch | controller → switch | 스위치 설정, flow entry 수정 |
| Asynchronous         | switch → controller | 이벤트나 상태 변화 보고      |
| Symmetric            | 양방향              | 기타 관리 메시지             |

OpenFlow API와 OpenFlow protocol은 구분해야 한다.  
API는 어떤 forwarding action을 지정할 수 있는지에 가깝고, protocol은 controller와 switch가 실제 메시지를 교환하는 방식이다.

### 21-1. Controller-to-switch 메시지

| 메시지       | 역할                                                    |
| ------------ | ------------------------------------------------------- |
| features     | switch 기능 조회                                        |
| configure    | switch 설정 조회/변경                                   |
| modify-state | flow table entry 추가, 삭제, 수정                       |
| packet-out   | controller가 특정 패킷을 특정 switch port로 내보내게 함 |

### 21-2. Switch-to-controller 메시지

| 메시지       | 역할                                               |
| ------------ | -------------------------------------------------- |
| packet-in    | switch가 처리할 수 없는 패킷을 controller에게 전달 |
| flow-removed | flow table entry가 삭제되었음을 알림               |
| port status  | port 상태 변화 알림                                |

실제 네트워크 운영자는 OpenFlow 메시지를 직접 손으로 작성하지 않는다.  
보통 컨트롤러가 제공하는 더 높은 수준의 추상화와 도구를 사용한다.

---

## 22. SDN에서 링크 장애가 발생했을 때 흐름

SDN에서 링크 장애가 발생하면 control/data plane이 다음처럼 상호작용한다.

```text
1. Switch가 link failure 감지
2. Switch가 OpenFlow port status 메시지로 controller에게 알림
3. Controller가 network state를 갱신
4. Routing application이 link state 변화 이벤트를 받음
5. Routing application이 network graph를 보고 새 경로 계산
6. Controller가 필요한 switch들의 flow table을 새로 계산
7. OpenFlow를 통해 새 flow rule을 switch에 설치
```

이 흐름에서 중요한 점은, 스위치가 모든 경로 계산을 직접 하는 것이 아니라 컨트롤러와 application이 네트워크 상태를 보고 계산한다는 것이다.

---

## 23. ODL과 ONOS Controller

SDN controller의 구현 예로 ODL과 ONOS가 있다.

### 23-1. OpenDaylight, ODL

ODL은 Service Abstraction Layer, 즉 SAL을 중심으로 내부/외부 application과 service를 연결한다.  
OpenFlow, NETCONF, SNMP, OVSDB 같은 southbound protocol을 사용할 수 있고, REST/RESTCONF/NETCONF API를 제공한다.

### 23-2. ONOS

ONOS는 분산 core를 강조하는 SDN controller이다.  
신뢰성, 복제, 성능 확장에 초점을 둔다.

ONOS의 intent framework는 “어떻게 할지”보다 “무엇을 원하는지”를 높은 수준에서 표현하게 한다.

```text
낮은 수준: switch1의 port2에 flow rule을 넣어라
높은 수준: host A와 host B가 통신 가능하게 하라
```

SDN의 중요한 과제는 controller를 안정적이고 확장 가능하며 안전한 분산 시스템으로 만드는 것이다.

---

## 24. ICMP

ICMP는 **Internet Control Message Protocol**이다.  
호스트와 라우터가 네트워크 계층 정보를 주고받기 위해 사용한다.

ICMP의 대표 역할은 두 가지이다.

```text
1. 오류 보고
   예: destination unreachable, TTL expired, bad IP header

2. 진단
   예: ping의 echo request / echo reply
```

ICMP는 IP 위에서 동작한다.  
즉, ICMP 메시지는 IP datagram 안에 실려 전달된다.

대표 type/code는 다음 정도를 기억하면 된다.

| Type | Code | 의미                         |
| ---- | ---- | ---------------------------- |
| 0    | 0    | Echo reply, ping 응답        |
| 3    | 3    | Destination port unreachable |
| 8    | 0    | Echo request, ping 요청      |
| 11   | 0    | TTL expired                  |
| 12   | 0    | Bad IP header                |

---

## 25. Traceroute와 ICMP

Traceroute는 TTL과 ICMP를 이용해 목적지까지 가는 경로를 추적한다.

동작 흐름은 다음과 같다.

```text
1. 출발지가 TTL=1인 UDP segment들을 보냄
2. 첫 번째 라우터에서 TTL이 0이 됨
3. 라우터가 datagram을 버리고 ICMP TTL expired 메시지를 보냄
4. 출발지는 이 응답으로 첫 번째 라우터와 RTT를 기록
5. 다음에는 TTL=2로 보냄
6. 두 번째 라우터가 ICMP TTL expired를 보냄
7. 이 과정을 TTL을 늘려가며 반복
```

목적지 host에 UDP segment가 도착하면, 보통 해당 port가 열려 있지 않기 때문에 destination은 ICMP port unreachable 메시지를 보낸다.  
출발지는 이 메시지를 보고 traceroute를 멈춘다.

```text
TTL=1 → 1번째 router 확인
TTL=2 → 2번째 router 확인
TTL=3 → 3번째 router 확인
...
목적지 도착 → ICMP port unreachable → 종료
```

---

## 26. Network Management

Network management는 네트워크 장비와 서비스를 모니터링하고, 설정하고, 분석하고, 제어하는 활동이다.

관리 대상은 단순히 라우터 하나가 아니라 수많은 hardware/software component로 구성된 AS 전체일 수 있다.

Network management의 기본 구성은 다음과 같다.

| 요소                       | 의미                                       |
| -------------------------- | ------------------------------------------ |
| Managing server/controller | 관리자가 사용하는 서버 또는 컨트롤러       |
| Managed device             | 관리 대상 장비, 예: router, switch         |
| Agent                      | 장비 안에서 관리 정보를 제공하는 구성 요소 |
| Data                       | 설정 정보, 운영 상태, 통계 정보            |
| Management protocol        | server와 device가 정보를 주고받는 규칙     |

```text
[Managing Server]
       │ request / configuration / query
       v
[Managed Device + Agent + Data]
       │ response / trap / notification
       v
[Managing Server]
```

네트워크 관리 방식은 CLI, SNMP/MIB, NETCONF/YANG으로 발전해 왔다.

---

## 27. SNMP와 MIB

SNMP는 **Simple Network Management Protocol**이다.  
관리 서버가 장비 상태를 조회하거나 설정값을 변경하고, 장비가 예외 상황을 관리자에게 알리는 데 사용된다.

SNMP에는 두 가지 동작 방식이 있다.

| 방식             | 의미                                             |
| ---------------- | ------------------------------------------------ |
| Request/Response | manager가 agent에게 정보를 요청하고 agent가 응답 |
| Trap             | agent가 예외 상황을 manager에게 먼저 알림        |

SNMP 메시지 종류는 다음과 같다.

| 메시지         | 역할                           |
| -------------- | ------------------------------ |
| GetRequest     | 특정 관리 정보 요청            |
| GetNextRequest | 목록에서 다음 정보 요청        |
| GetBulkRequest | 여러 정보를 한 번에 요청       |
| SetRequest     | MIB 값을 설정                  |
| Response       | 요청에 대한 응답               |
| Trap           | 예외 이벤트를 manager에게 알림 |

MIB는 **Management Information Base**이다.  
관리 가능한 장비 상태와 통계 값들을 구조화한 데이터 모음이다.

예를 들어 UDP 관련 MIB에는 다음 같은 정보가 있을 수 있다.

```text
UDPInDatagrams   : 수신되어 전달된 UDP datagram 수
UDPNoPorts       : 해당 port가 없어 전달되지 못한 datagram 수
UDPOutDatagrams  : 전송된 UDP datagram 수
```

SNMP는 전통적으로 장비의 운영 상태를 조회하고 모니터링하는 데 많이 쓰인다.

---

## 28. NETCONF와 YANG

NETCONF는 여러 네트워크 장비의 설정을 더 구조적이고 일관되게 관리하기 위한 프로토콜이다.  
SNMP가 장비 상태 조회와 단순 관리에 초점이 있다면, NETCONF/YANG은 configuration management에 더 강하다.

NETCONF의 주요 특징은 다음과 같다.

```text
- 관리 서버와 managed device 사이에서 동작
- 설정 조회, 변경, 활성화 가능
- 여러 장비에 대한 atomic commit 지원 가능
- 운영 상태와 통계 조회 가능
- notification 구독 가능
- RPC 방식 사용
- 메시지는 XML로 인코딩
- 안전하고 신뢰성 있는 전송 방식 사용
```

NETCONF 세션 흐름은 다음과 같다.

```text
1. <hello>로 session initiation 및 capability exchange
2. <rpc> 요청 전송
3. <rpc-reply> 응답 수신
4. 필요 시 <notification> 수신
5. <close-session>으로 종료
```

대표 NETCONF operation은 다음과 같다.

| Operation             | 의미                                   |
| --------------------- | -------------------------------------- |
| <get-config>          | 특정 configuration 조회                |
| <get>                 | configuration + operational state 조회 |
| <edit-config>         | 장비 설정 변경                         |
| <lock>, <unlock>      | configuration datastore 잠금/해제      |
| <create-subscription> | event notification 구독                |

YANG은 NETCONF에서 사용하는 data modeling language이다.  
즉, 장비 설정 데이터의 구조, 문법, 의미, 제약 조건을 정의한다.

```text
YANG = 데이터 모델 정의
NETCONF = 그 모델에 맞는 설정/조회/변경 메시지를 주고받는 프로토콜
```

YANG을 사용하면 설정 데이터가 일관된 구조를 갖게 되고, 잘못된 configuration을 줄일 수 있다.

---

## 29. Chapter 5 핵심 비교 모음

### 29-1. Forwarding vs Routing

| 구분 | Forwarding              | Routing               |
| ---- | ----------------------- | --------------------- |
| 평면 | Data Plane              | Control Plane         |
| 위치 | 라우터 내부의 빠른 처리 | 경로 계산 과정        |
| 역할 | 입력 포트 → 출력 포트   | 목적지까지 경로 결정  |
| 결과 | 패킷 전달               | forwarding table 생성 |

### 29-2. Per-router vs SDN

| 구분      | Per-router           | SDN                                |
| --------- | -------------------- | ---------------------------------- |
| 제어 방식 | 라우터마다 분산 제어 | 논리적 중앙 제어                   |
| 계산 주체 | 각 라우터            | SDN controller                     |
| 장비 성격 | 라우터가 똑똑함      | switch는 단순, controller가 똑똑함 |
| 장점      | 전통적이고 분산적    | 관리와 정책 적용 쉬움              |

### 29-3. LS vs DV

| 구분          | Link State                    | Distance Vector                  |
| ------------- | ----------------------------- | -------------------------------- |
| 정보          | 전체 지도                     | 이웃 정보                        |
| 대표 알고리즘 | Dijkstra                      | Bellman-Ford                     |
| 문제          | oscillation 가능              | count-to-infinity 가능           |
| 감각          | 전체 지도 보고 최단 경로 계산 | 이웃에게 물어보며 최소 비용 계산 |

### 29-4. OSPF vs BGP

| 구분      | OSPF       | BGP                |
| --------- | ---------- | ------------------ |
| 범위      | AS 내부    | AS 사이            |
| 유형      | Link State | Path Vector        |
| 우선 기준 | 성능, 비용 | 정책, reachability |
| 사용 위치 | intra-AS   | inter-AS           |

### 29-5. SNMP vs NETCONF/YANG

| 구분        | SNMP/MIB               | NETCONF/YANG                    |
| ----------- | ---------------------- | ------------------------------- |
| 중심        | 상태 조회, 모니터링    | 설정 관리, 구조적 configuration |
| 데이터 모델 | MIB                    | YANG                            |
| 통신 방식   | request/response, trap | RPC, XML 메시지                 |
| 감각        | 장비 상태를 물어봄     | 장비 설정을 모델 기반으로 관리  |

---

## 30. 생소한 용어 한 줄 정리

| 용어               | 한 줄 의미                                                             |
| ------------------ | ---------------------------------------------------------------------- |
| Control Plane      | forwarding table을 어떻게 만들지 결정하는 영역                         |
| Data Plane         | 만들어진 forwarding table을 보고 패킷을 전달하는 영역                  |
| Path               | 패킷이 지나가는 라우터들의 순서                                        |
| Link Cost          | 특정 링크를 사용하는 비용 또는 부담                                    |
| Link State         | 전체 네트워크 연결 상태를 알고 계산하는 방식                           |
| Distance Vector    | 이웃이 알려준 거리 정보를 바탕으로 계산하는 방식                       |
| Dijkstra           | 한 출발지에서 모든 목적지까지 최소 비용 경로 계산                      |
| Bellman-Ford       | 이웃을 거치는 비용 중 최소값을 반복 계산                               |
| Oscillation        | 경로와 비용이 반복적으로 흔들리는 현상                                 |
| Count-to-infinity  | 나쁜 경로 정보가 천천히 수렴하는 DV 문제                               |
| Black-holing       | 잘못된 낮은 비용 광고 때문에 트래픽이 특정 라우터로 빨려 들어가는 현상 |
| AS                 | 하나의 관리 주체가 운영하는 네트워크 묶음                              |
| Intra-AS           | 같은 AS 내부 라우팅                                                    |
| Inter-AS           | 서로 다른 AS 사이 라우팅                                               |
| Gateway Router     | 다른 AS와 연결되는 AS의 가장자리 라우터                                |
| OSPF               | AS 내부에서 쓰는 대표 link-state 라우팅 프로토콜                       |
| BGP                | AS 사이에서 쓰는 대표 path-vector 라우팅 프로토콜                      |
| AS-PATH            | 목적지까지 거쳐야 하는 AS들의 목록                                     |
| NEXT-HOP           | 외부 목적지로 나가기 위해 내부에서 향해야 할 gateway                   |
| Hot Potato Routing | 자기 AS 안에서 가장 가까운 출구로 빨리 내보내는 방식                   |
| SDN                | controller가 forwarding rule을 계산·설치하는 네트워크 구조             |
| Northbound API     | controller와 network control application 사이 API                      |
| Southbound API     | controller와 switch 사이 API                                           |
| OpenFlow           | SDN controller와 switch가 통신하는 대표 프로토콜                       |
| ICMP               | 네트워크 계층 오류 보고와 진단 프로토콜                                |
| Traceroute         | TTL 만료와 ICMP 응답으로 경로를 추적하는 도구                          |
| SNMP               | 장비 상태 조회와 이벤트 알림을 위한 관리 프로토콜                      |
| MIB                | SNMP에서 관리 가능한 장비 정보의 구조화된 모음                         |
| NETCONF            | 장비 설정을 구조적으로 조회·변경하는 관리 프로토콜                     |
| YANG               | NETCONF 설정 데이터의 구조와 제약을 정의하는 모델 언어                 |

---

## 31. 마지막 암기 문장

Chapter 5는 네트워크 계층의 control plane을 설명하는 장이다.  
Control plane은 routing을 통해 forwarding table을 만들고, data plane은 그 table을 이용해 packet을 forwarding한다.  
전통적 방식에서는 각 라우터가 routing algorithm을 수행하지만, SDN에서는 controller가 forwarding rule을 계산해 switch에 설치한다.  
Routing algorithm은 크게 전체 지도를 보는 Link State와 이웃 정보를 이용하는 Distance Vector로 나뉘며, 실제 인터넷에서는 AS 단위로 나누어 내부는 OSPF, 외부는 BGP로 라우팅한다.  
BGP는 단순 최단 경로보다 정책을 중요하게 보며, SDN은 중앙 제어와 flow 기반 forwarding으로 더 유연한 네트워크 제어를 가능하게 한다.  
ICMP는 오류 보고와 진단을 담당하고, SNMP와 NETCONF/YANG은 네트워크 장비를 모니터링하고 설정하기 위한 관리 기술이다.

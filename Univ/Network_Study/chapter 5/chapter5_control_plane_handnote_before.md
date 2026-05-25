# Chapter 5 Network Layer: Control Plane 손필기 전사용 노트

## 목적

- 시험 직전 복습 가능
- 손으로 전사하기 쉬운 구조
- 교안 + 직접 필기 + 수업식 강조 포인트 통합
- 생소한 개념은 따로 풀어서 설명
- 그림은 가능한 한 ASCII 와이어프레임으로 변환

## 이 노트의 사용 방식

- 먼저 `0. 핵심 문장`을 외운다.
- 그 다음 개념별 본문을 읽는다.
- 비교표와 와이어프레임은 시험 직전 다시 본다.
- 수식은 “공식 자체”보다 “무엇을 비교하는 공식인지”를 이해한다.

---

# 0. 먼저 외울 핵심 문장

1. 네트워크 계층은 데이터 평면과 컨트롤 평면으로 나뉜다.
2. 데이터 평면은 포워딩을 담당한다.
3. 컨트롤 평면은 라우팅을 담당한다.
4. 포워딩은 “들어온 패킷을 어느 출력 포트로 보낼지” 결정한다.
5. 라우팅은 “목적지까지 어떤 경로를 사용할지” 결정한다.
6. 전통적 방식은 각 라우터가 스스로 라우팅 알고리즘을 수행한다.
7. SDN 방식은 원격 컨트롤러가 forwarding table을 계산하고 설치한다.
8. 라우팅 프로토콜의 목표는 좋은 path를 찾는 것이다.
9. 좋은 path란 보통 비용이 낮고, 빠르고, 혼잡이 적은 경로이다.
10. Link State 알고리즘은 전체 네트워크 지도를 알고 계산한다.
11. Distance Vector 알고리즘은 이웃이 알려준 거리 정보를 바탕으로 계산한다.
12. Dijkstra 알고리즘은 한 출발지에서 모든 목적지까지의 최소 비용 경로를 구한다.
13. Bellman-Ford 식은 “이웃을 거쳐 가는 비용 중 최소값”을 고른다.
14. LS는 빠르게 계산되지만 link cost가 트래픽에 따라 바뀌면 oscillation이 생길 수 있다.
15. DV는 분산적이고 단순하지만 count-to-infinity 문제가 생길 수 있다.
16. 실제 인터넷은 너무 커서 하나의 평평한 네트워크처럼 라우팅할 수 없다.
17. 그래서 인터넷은 AS, 즉 Autonomous System 단위로 나누어 관리한다.
18. AS 내부 라우팅은 intra-AS, AS 사이 라우팅은 inter-AS 라우팅이다.
19. OSPF는 대표적인 intra-AS 링크 상태 라우팅 프로토콜이다.
20. BGP는 대표적인 inter-AS 라우팅 프로토콜이며 인터넷을 묶는 접착제 역할을 한다.
21. BGP는 단순 최단 경로보다 정책을 더 중요하게 고려한다.
22. Hot potato routing은 내 AS 안에서 가장 빨리 밖으로 던지는 방식이다.
23. SDN은 제어 기능을 라우터 내부가 아니라 컨트롤러 쪽으로 분리한다.
24. OpenFlow는 SDN 컨트롤러와 스위치 사이에서 메시지를 주고받는 대표 프로토콜이다.
25. ICMP는 네트워크 계층의 오류 보고와 진단을 위한 프로토콜이다.
26. ping은 ICMP echo request/reply를 사용한다.
27. traceroute는 TTL 만료와 ICMP 응답을 이용해 경로를 추적한다.
28. SNMP는 장비 상태를 조회하거나 이벤트를 전달받는 전통적 관리 프로토콜이다.
29. NETCONF/YANG은 여러 장비의 설정을 더 구조적이고 일관되게 관리하기 위한 방식이다.
30. Chapter 5의 큰 질문은 “라우터들이 어떻게 경로를 알고, 누가 그 경로를 정하는가?”이다.

---

# 1. 컨트롤 플레인의 큰 그림 (p5-2 ~ p5-7)

## 1-1. 정의 / 큰 그림

- Chapter 5의 핵심은 네트워크 계층의 **Control Plane**이다.
- 직접 필기식으로 쓰면 다음과 같다.
  - “컨트롤 플레인 = 실제로 라우팅을 어떻게 할 것인가”
  - “라우팅 알고리즘, 인터넷 구현 방법, SDN, 관리 프로토콜까지 다룸”

네트워크 계층 기능은 크게 둘로 나뉜다.

| 구분 | 역할 | 감각적 표현 |
|---|---|---|
| Data Plane | forwarding | 들어온 패킷을 어느 포트로 내보낼지 처리 |
| Control Plane | routing | 목적지까지 어떤 경로를 쓸지 결정 |

- **포워딩**은 각 라우터 내부에서 매우 빠르게 일어난다.
- **라우팅**은 forwarding table이 어떻게 만들어지는지를 다룬다.
- 즉, forwarding table을 보고 움직이는 것이 데이터 평면이고,
- forwarding table을 만들어내는 것이 컨트롤 평면이다.

## 1-2. 손필기식 표현

- 포워딩
  - “패킷이 들어왔을 때, 어디로 내보낼지”
  - 이미 만들어진 표를 보고 빠르게 처리

- 라우팅
  - “목적지까지 길을 어떻게 정할지”
  - 라우팅 알고리즘이 forwarding table을 만들어줌

- 이 장의 큰 질문
  - “라우터가 경로를 어떻게 알고 있을까?”
  - “그 경로는 각 라우터가 직접 계산할까?”
  - “아니면 중앙 컨트롤러가 계산해줄까?”

## 1-3. 시험 포인트

- forwarding과 routing 구분은 반드시 정확히 써야 한다.
- forwarding은 data plane이다.
- routing은 control plane이다.
- per-router control과 SDN control의 차이를 비교할 수 있어야 한다.

## 1-4. ASCII 와이어프레임

```text
네트워크 계층 기능 구분

[Control Plane]
  - 라우팅 알고리즘
  - 경로 계산
  - forwarding table 생성
          |
          v
[Data Plane]
  - forwarding table 참조
  - 입력 포트 -> 출력 포트
  - 패킷을 빠르게 이동
```

```text
패킷 처리 감각

패킷 도착
   |
   v
헤더 값 확인
   |
   v
forwarding table 조회
   |
   v
출력 포트 선택
   |
   v
다음 라우터로 전달
```

---

# 2. Per-router Control Plane vs SDN Control Plane (p5-4 ~ p5-6, p5-65 ~ p5-68)

## 2-1. 정의 / 큰 그림

컨트롤 평면을 구성하는 방식은 크게 두 가지이다.

1. Per-router control plane
2. Logically centralized control plane, 즉 SDN

## 2-2. Per-router Control Plane

- 전통적인 방식이다.
- 각 라우터가 자기 안에서 라우팅 알고리즘을 수행한다.
- 라우터마다 control plane 기능이 들어 있다.
- 라우터들이 서로 정보를 교환한다.
- 그 결과 각자 forwarding table을 만든다.

손필기식 표현:

- “라우터마다 컨트롤 플레인을 제어”
- “각각의 모든 라우터가 동작”
- “라우팅 알고리즘을 통해 forwarding table 값을 가짐”

## 2-3. SDN Control Plane

- SDN은 Software-Defined Networking이다.
- 라우터나 스위치 안에서 모든 제어를 하지 않는다.
- 원격 컨트롤러가 경로와 forwarding table을 계산한다.
- 스위치/라우터는 컨트롤러가 내려준 규칙을 따라 빠르게 forwarding한다.

손필기식 표현:

- “리모트 컨트롤러가 존재”
- “계산한 것을 control agent가 받아옴”
- “스위치는 단순하게 forwarding에 집중”

## 2-4. 비교표

| 구분 | Per-router Control | SDN Control |
|---|---|---|
| 제어 위치 | 각 라우터 내부 | 논리적 중앙 컨트롤러 |
| forwarding table 계산 | 라우터들이 분산 계산 | 컨트롤러가 계산 후 설치 |
| 장점 | 인터넷의 전통적 방식, 분산적 | 관리 쉬움, 정책 적용 쉬움 |
| 단점 | 설정 복잡, 전체 제어 어려움 | 컨트롤러 신뢰성/확장성 필요 |
| 감각 | 각자 알아서 길 찾기 | 관제센터가 길 지시 |

## 2-5. 왜 SDN이 나왔는가?

전통적 라우팅은 link weight를 조정하는 식으로 간접 제어를 한다.

예를 들어 운영자가 원하는 것이 다음과 같다고 하자.

- u에서 z로 가는 트래픽을 특정 경로로 보내고 싶다.
- 두 경로로 트래픽을 나누고 싶다.
- 빨간 트래픽과 파란 트래픽을 다르게 보내고 싶다.

전통적인 목적지 기반 라우팅에서는 이것이 어렵다.

SDN에서는 flow 기반으로 더 세밀하게 제어할 수 있다.

## 2-6. ASCII 와이어프레임

```text
Per-router control

[Router A] <----라우팅 정보 교환----> [Router B]
   |                                      |
   v                                      v
자기 forwarding table 계산          자기 forwarding table 계산

특징: 각 라우터가 스스로 계산한다.
```

```text
SDN control

            [Remote Controller]
          /        |        |       \
         v         v        v        v
     [Switch1] [Switch2] [Switch3] [Switch4]

특징: 컨트롤러가 계산하고 스위치에 규칙을 설치한다.
```

---

# 3. 라우팅 프로토콜과 그래프 추상화 (p5-8 ~ p5-10)

## 3-1. 라우팅 프로토콜의 목표

- 라우팅 프로토콜의 목표는 좋은 path를 정하는 것이다.
- path는 출발 호스트에서 목적지 호스트까지 패킷이 지나가는 라우터들의 순서이다.
- 좋은 path는 상황에 따라 다르게 정의된다.

좋은 경로의 기준:

- 비용이 낮은 경로
- 빠른 경로
- 혼잡이 적은 경로
- 정책상 허용되는 경로

손필기식 표현:

- “path = 패킷을 잘 전달하는 통로”
- “cost = 비용, 적을수록 좋음”
- “최소 비용 경로를 찾는 것이 기본 감각”

## 3-2. 그래프 추상화

라우팅 문제는 그래프로 표현할 수 있다.

- G = (N, E)
- N = 노드 집합, 즉 라우터 집합
- E = 링크 집합, 즉 라우터 사이 연결
- c(a,b) = a와 b 사이 직접 링크 비용
- 직접 연결이 없으면 비용은 ∞로 본다.

## 3-3. link cost의 의미

link cost는 네트워크 운영자가 정할 수 있다.

예시:

- 모든 링크 비용을 1로 둠
- 대역폭이 큰 링크는 비용을 낮게 둠
- 혼잡한 링크는 비용을 높게 둠
- 지연이 큰 링크는 비용을 높게 둠

중요한 점:

- 비용이 낮다고 항상 물리적 거리가 짧다는 뜻은 아니다.
- 비용은 운영 정책과 성능 기준이 반영된 값이다.

## 3-4. 라우팅 알고리즘 분류

라우팅 알고리즘은 두 가지 기준으로 나눌 수 있다.

### 기준 1: 정보의 범위

| 구분 | 의미 | 대표 알고리즘 |
|---|---|---|
| Global | 모든 라우터가 전체 topology와 link cost를 앎 | Link State |
| Decentralized | 각 라우터가 이웃 정보에서 출발해 반복적으로 학습 | Distance Vector |

### 기준 2: 경로 변화 속도

| 구분 | 의미 |
|---|---|
| Static | 경로가 천천히 바뀜 |
| Dynamic | 경로가 자주 바뀔 수 있음 |

## 3-5. 손필기식 표현 보강

- Global 방식
  - “전체 지도를 들고 계산”
  - 모든 링크 상태를 알고 있음

- Decentralized 방식
  - “내 이웃이 알려준 정보로 조금씩 계산”
  - 처음에는 직접 연결된 이웃 비용만 앎

## 3-6. ASCII 와이어프레임

```text
그래프 추상화

라우터 = 노드
링크   = 간선
비용   = 간선의 weight

      2
  A ----- B
  |       |
5 |       | 1
  |       |
  C ----- D
      3

A에서 D까지 비용 후보
A-B-D = 2 + 1 = 3
A-C-D = 5 + 3 = 8
=> 최소 비용 경로는 A-B-D
```

---

# 4. Link State Routing과 Dijkstra 알고리즘 (p5-11 ~ p5-18)

## 4-1. 정의 / 큰 그림

Link State 라우팅은 모든 라우터가 전체 네트워크 topology와 link cost 정보를 알고 있다고 가정한다.

- 각 라우터는 자신의 link state 정보를 네트워크 전체에 알린다.
- 모든 라우터는 같은 네트워크 지도를 갖게 된다.
- 각 라우터는 Dijkstra 알고리즘으로 자신 기준 최단 경로를 계산한다.
- 계산 결과로 forwarding table을 만든다.

손필기식 표현:

- “링크 코스트 등을 알고 있을 때”
- “특정 노드로부터 다른 노드까지의 최단 거리를 계산”
- “전체 지도를 가지고 각자 자기 출발지 기준으로 계산”

## 4-2. Dijkstra 알고리즘의 핵심 변수

| 기호 | 의미 | 손필기식 감각 |
|---|---|---|
| c(x,y) | x에서 y로 직접 가는 link cost | 직접 연결 비용 |
| D(v) | 현재까지 알고 있는 출발지에서 v까지의 최소 비용 추정치 | 지금까지의 최선값 |
| p(v) | v로 가기 직전 노드 | 경로 복원용 이전 노드 |
| N' | 최소 비용 경로가 확정된 노드 집합 | 확정된 애들 모음 |

## 4-3. 알고리즘 흐름

1. 출발 노드 u를 N'에 넣는다.
2. u와 직접 연결된 노드의 D 값을 초기화한다.
3. 직접 연결이 없으면 D 값을 ∞로 둔다.
4. 아직 확정되지 않은 노드 중 D 값이 가장 작은 노드 w를 고른다.
5. w를 N'에 넣어 확정한다.
6. w의 이웃 노드들에 대해 더 싼 경로가 생겼는지 갱신한다.
7. 모든 노드가 확정될 때까지 반복한다.

## 4-4. 핵심 갱신식

```text
D(v) = min( D(v), D(w) + c(w,v) )
```

의미:

- 기존에 알고 있던 v까지의 비용을 유지할지,
- 새로 확정된 w를 거쳐 v로 가는 비용을 사용할지 비교한다.

손필기식 표현:

- “직접 연결이 안 되어 있으면 ∞를 씀”
- “근접한 노드만 탐색하면서 거리 정보를 갱신”
- “집합 N'은 최단 경로가 확정된 노드들의 모음”

## 4-5. Dijkstra 표를 읽는 법

Dijkstra 예시 표에서는 보통 다음을 본다.

- Step: 몇 번째 반복인지
- N': 확정된 노드 집합
- D(v), p(v): v까지의 현재 최소 비용과 직전 노드
- 가장 작은 D 값을 가진 노드를 다음에 확정
- p(v)를 거꾸로 추적하면 최단 경로 트리가 나온다.

## 4-6. 예시 감각

```text
출발지 u

초기 상태:
N' = {u}
D(이웃) = u에서 직접 가는 비용
D(비이웃) = ∞

반복:
1. 아직 확정 안 된 노드 중 D가 제일 작은 노드 선택
2. 선택한 노드를 확정
3. 그 노드를 거쳐 가면 더 싸지는지 확인
```

## 4-7. 복잡도

- 단순 구현에서는 O(n²)
- 더 효율적인 자료구조를 쓰면 O(n log n)까지 가능
- n은 라우터 수이다.

메시지 복잡도:

- 각 라우터가 자신의 link state 정보를 다른 라우터에게 알려야 한다.
- 전체적으로 많은 link state advertisement가 필요하다.

## 4-8. Oscillation 문제

Link cost가 트래픽 양에 따라 바뀌면 문제가 생길 수 있다.

상황:

1. 어떤 경로가 싸다고 판단되어 트래픽이 몰린다.
2. 트래픽이 몰리면 그 링크의 비용이 올라간다.
3. 그러면 다른 경로가 싸다고 판단된다.
4. 트래픽이 다른 경로로 이동한다.
5. 다시 비용이 바뀐다.
6. 경로가 계속 흔들릴 수 있다.

이것이 route oscillation이다.

손필기식 표현:

- “라우팅 경로에 따라 트래픽이 몰리면 링크 비용이 변할 수 있음”
- “비용이 트래픽 의존적이면 경로가 출렁일 수 있음”

## 4-9. 시험 포인트

- Dijkstra는 Link State 알고리즘이다.
- 전체 topology를 알고 계산한다.
- D(v), p(v), N'의 의미를 설명할 수 있어야 한다.
- `D(v) = min(D(v), D(w)+c(w,v))`의 의미를 말할 수 있어야 한다.
- link cost가 traffic-dependent이면 oscillation이 생길 수 있다.

## 4-10. ASCII 와이어프레임

```text
Dijkstra 갱신 구조

현재 확정 집합 N'
+-------------------+
| u, x, y ...       |
+-------------------+
          |
          v
아직 확정 안 된 노드 중 D 값 최소 선택
          |
          v
선택한 노드 w 확정
          |
          v
w의 이웃 v에 대해 갱신
D(v) = min(기존 비용, w까지 비용 + w-v 링크 비용)
```

```text
Oscillation 감각

경로 A가 싸다
   ↓
트래픽이 A로 몰림
   ↓
A의 비용 증가
   ↓
경로 B가 싸짐
   ↓
트래픽이 B로 이동
   ↓
B의 비용 증가
   ↓
다시 A가 싸짐
   ↓
경로가 계속 흔들림
```

---

# 5. Distance Vector Routing과 Bellman-Ford 알고리즘 (p5-19 ~ p5-39)

## 5-1. 정의 / 큰 그림

Distance Vector 라우팅은 전체 네트워크 지도를 처음부터 알고 계산하지 않는다.

- 각 라우터는 자신과 직접 연결된 이웃 비용만 안다.
- 각 라우터는 자신의 distance vector를 이웃에게 보낸다.
- 이웃이 알려준 정보를 이용해 자신의 거리 정보를 갱신한다.
- 이 과정이 반복되며 정보가 네트워크 전체로 퍼진다.

손필기식 표현:

- “벨만 포드 알고리즘 주로 이용”
- “결국 근처에 있는 애들 중 비용이 가장 적은 것”
- “계속 전해서 반복반복반복”
- “그때그때 이웃에 따라 바뀌기 때문에 반복적이고 비동기적”

## 5-2. Bellman-Ford 식

```text
D_x(y) = min_v { c_x,v + D_v(y) }
```

의미:

- x에서 y까지 가는 최소 비용을 구하고 싶다.
- x의 이웃 v들을 하나씩 본다.
- x에서 v까지 직접 가는 비용 `c_x,v`와
- v가 알고 있는 y까지 비용 `D_v(y)`를 더한다.
- 그중 가장 작은 값을 고른다.

## 5-3. 식을 말로 풀기

```text
x가 y로 가는 가장 좋은 길
= x의 이웃 중 하나 v를 먼저 거쳐서 가는 길 중
  전체 비용이 가장 작은 길
```

즉, x는 전체 지도를 몰라도 된다.

- “v야, 너 y까지 얼마에 갈 수 있어?”
- “그럼 내가 너한테 가는 비용까지 더하면 얼마지?”
- “이웃들 중 가장 싼 애를 next hop으로 고르자.”

## 5-4. Distance Vector 알고리즘 특징

| 특징 | 설명 |
|---|---|
| iterative | 변화가 있을 때마다 반복 계산 |
| asynchronous | 모든 라우터가 동시에 움직일 필요 없음 |
| distributed | 중앙 계산기가 없음 |
| self-stopping | 값이 안 바뀌면 더 이상 알리지 않음 |

업데이트가 발생하는 경우:

- local link cost가 바뀜
- 이웃에게서 새로운 DV 메시지를 받음

## 5-5. 정보 확산 감각

Distance Vector는 정보가 한 번에 전체로 퍼지지 않는다.

```text
t = 0 : 정보가 자기 자신에게만 있음
t = 1 : 이웃 1 hop까지 영향
t = 2 : 2 hop까지 영향
t = 3 : 3 hop까지 영향
...
```

즉, 정보가 파도처럼 퍼진다.

## 5-6. 좋은 소식은 빠르게 퍼진다

링크 비용이 낮아졌다고 하자.

- y가 link cost 감소를 감지한다.
- y는 자신의 DV를 갱신한다.
- 이웃에게 “더 싸게 갈 수 있음”을 알린다.
- 이웃도 더 좋은 경로를 빠르게 반영한다.

이것을 “good news travels fast”라고 한다.

## 5-7. 나쁜 소식은 느리게 퍼진다

링크 비용이 크게 증가하거나 경로가 끊기면 문제가 생긴다.

- y는 직접 경로가 나빠졌다는 것을 안다.
- 그런데 z가 “나는 x까지 싸게 갈 수 있어”라고 말할 수 있다.
- 사실 z의 경로도 y를 거치는 경로일 수 있다.
- y와 z가 서로를 믿고 비용을 조금씩 올린다.
- 비용이 6, 7, 8, 9 ... 식으로 천천히 증가한다.

이것이 count-to-infinity 문제이다.

## 5-8. Count-to-infinity 감각

```text
원래:
y -> x 직접 비용이 작음
z -> x는 y를 거쳐 감

문제 발생:
y -> x 비용이 갑자기 커짐

그런데 z가 말함:
"나는 x까지 5로 갈 수 있어"

실제로는?
z도 y를 거쳐 x로 가는 중이었음

결과:
y는 z를 믿음
z는 y를 믿음
서로 잘못된 경로를 믿으며 비용이 조금씩 증가
```

## 5-9. 시험 포인트

- Distance Vector는 Bellman-Ford 기반이다.
- 각 노드는 이웃에게 자신의 거리 벡터를 보낸다.
- 전체 topology를 몰라도 계산이 진행된다.
- good news travels fast와 bad news travels slow를 구분한다.
- count-to-infinity 문제를 말로 설명할 수 있어야 한다.

## 5-10. ASCII 와이어프레임

```text
Distance Vector 업데이트

[라우터 x]
  |
  | 이웃 v들의 DV 수신
  v
각 목적지 y에 대해 계산

D_x(y) = min_v { c_x,v + D_v(y) }

  |
  v
내 DV가 바뀌면 이웃에게 알림
```

```text
Bellman-Ford 감각

목적지 y로 가고 싶음

이웃 A 통해 가기 = x-A 비용 + A가 아는 y 비용
이웃 B 통해 가기 = x-B 비용 + B가 아는 y 비용
이웃 C 통해 가기 = x-C 비용 + C가 아는 y 비용

=> 가장 작은 값을 선택
=> 선택된 이웃이 next hop
```

```text
Bad news travels slow

x 경로 나빠짐
   ↓
y가 z에게 물어봄
   ↓
z: "나 x까지 싸게 갈 수 있어"
   ↓
실은 z도 y를 거쳐 가는 중
   ↓
y와 z가 서로를 믿음
   ↓
비용이 6 -> 7 -> 8 -> 9 ...
   ↓
count-to-infinity
```

---

# 6. Link State vs Distance Vector 비교 (p5-40)

## 6-1. 큰 비교

| 구분 | Link State | Distance Vector |
|---|---|---|
| 대표 알고리즘 | Dijkstra | Bellman-Ford |
| 정보 범위 | 전체 topology | 이웃의 거리 정보 |
| 계산 방식 | 각 라우터가 전체 지도 기반 계산 | 이웃과 반복 교환 |
| 메시지 | link state 정보를 넓게 전파 | 이웃끼리 DV 교환 |
| 수렴 속도 | 비교적 빠른 편 | 상황에 따라 다름 |
| 문제 | oscillation 가능 | routing loop, count-to-infinity |
| 장애/악성 라우터 영향 | 잘못된 link cost 광고 가능 | 잘못된 DV가 다른 라우터 계산에 전파 가능 |

## 6-2. 이해 포인트

LS는 “전체 지도를 보고 각자 계산”이다.

DV는 “이웃 말을 듣고 내 표를 고침”이다.

그래서 DV는 한 라우터의 잘못된 정보가 다른 라우터의 table에 영향을 줄 수 있다.

## 6-3. 블랙홀링 개념

black-holing은 잘못된 라우터가 다음처럼 말하는 상황이다.

- “나는 모든 목적지까지 아주 낮은 비용으로 갈 수 있어.”

다른 라우터들이 이를 믿으면 트래픽이 그 라우터로 몰린다.

그 라우터가 실제로 전달하지 못하면 패킷이 사라지는 것처럼 보인다.

그래서 black hole이라는 표현을 쓴다.

## 6-4. 시험 포인트

- LS와 DV의 가장 큰 차이는 정보의 범위이다.
- LS는 global information이다.
- DV는 decentralized information이다.
- LS의 대표 알고리즘은 Dijkstra이다.
- DV의 대표 알고리즘은 Bellman-Ford이다.
- LS는 oscillation, DV는 count-to-infinity를 연결해서 외운다.

## 6-5. ASCII 비교

```text
Link State

모든 라우터가 전체 지도 보유
A: 전체 지도 -> A 기준 최단 경로 계산
B: 전체 지도 -> B 기준 최단 경로 계산
C: 전체 지도 -> C 기준 최단 경로 계산

Distance Vector

A <-> B <-> C
A는 B가 말한 비용을 듣고 갱신
B는 A, C가 말한 비용을 듣고 갱신
C는 B가 말한 비용을 듣고 갱신
```

---

# 7. 라우팅 확장성: AS, Intra-AS, Inter-AS (p5-41 ~ p5-46)

## 7-1. 왜 확장성 문제가 생기는가?

지금까지의 라우팅 설명은 이상화된 상황이다.

- 모든 라우터가 동일하다.
- 네트워크가 평평하다.
- 하나의 관리자가 전체 네트워크를 관리한다.

하지만 실제 인터넷은 그렇지 않다.

현실 문제:

- 목적지가 수십억 개이다.
- 모든 목적지를 routing table에 저장하기 어렵다.
- 모든 라우터가 모든 정보를 교환하면 링크가 감당하지 못한다.
- 인터넷은 여러 조직과 ISP가 연결된 network of networks이다.
- 각 조직은 자기 네트워크 정책을 직접 통제하고 싶어 한다.

## 7-2. Autonomous System, AS

AS는 하나의 관리 주체가 운영하는 네트워크 묶음이다.

- AS = Autonomous System
- domain이라고도 부른다.
- 예: 하나의 ISP, 큰 기업망, 대학망 등

손필기식 표현:

- “AS = domain 같은 것”
- “한 군데에서 모든 네트워크를 관리할 수 없기 때문에 나눈 단위”

## 7-3. Intra-AS와 Inter-AS

| 구분 | 의미 | 대표 예 |
|---|---|---|
| Intra-AS routing | 같은 AS 내부 라우팅 | OSPF, RIP, EIGRP |
| Inter-AS routing | 서로 다른 AS 사이 라우팅 | BGP |

- intra는 내부이다.
- inter는 외부와 외부 사이다.

## 7-4. Gateway Router

Gateway router는 AS의 가장자리에 있는 라우터이다.

역할:

- 자기 AS 내부 라우팅도 한다.
- 다른 AS와 연결되는 inter-AS 라우팅도 한다.
- 내부와 외부를 잇는 출입구 역할을 한다.

## 7-5. 외부 목적지로 가는 forwarding table 생성

AS1 내부 라우터가 AS 밖의 목적지로 패킷을 보내야 한다고 하자.

필요한 정보:

1. 목적지가 어느 AS를 통해 도달 가능한지
2. 그 정보를 AS 내부 라우터들에게 어떻게 전파할지
3. 내부 라우터가 어느 gateway router로 보낼지

즉, 외부 목적지에 대해서는 inter-AS와 intra-AS가 함께 forwarding table에 영향을 준다.

## 7-6. 시험 포인트

- 실제 인터넷은 flat network가 아니다.
- 확장성과 관리 자율성 때문에 AS로 나눈다.
- AS 내부는 intra-AS, AS 사이는 inter-AS이다.
- Gateway router는 두 영역을 연결한다.
- 외부 목적지 forwarding은 BGP 정보와 OSPF 같은 내부 라우팅 정보가 함께 필요하다.

## 7-7. ASCII 와이어프레임

```text
인터넷 구조 감각

        [AS 1]
     내부 라우터들
          |
     gateway router
          |
          | inter-AS routing
          |
     gateway router
     내부 라우터들
        [AS 2]
```

```text
외부 목적지 forwarding table 생성

BGP가 알려줌:
"목적지 X는 gateway G를 통해 갈 수 있음"

OSPF가 알려줌:
"내부에서 gateway G까지 가려면 interface 2 사용"

결과:
목적지 X -> interface 2
```

---

# 8. Intra-AS Routing과 OSPF (p5-46 ~ p5-48)

## 8-1. 대표적인 intra-AS 라우팅 프로토콜

| 프로토콜 | 기반 | 특징 |
|---|---|---|
| RIP | Distance Vector | 30초마다 DV 교환, 현재는 잘 쓰이지 않음 |
| EIGRP | DV 기반 | Cisco 중심으로 쓰였고 이후 공개됨 |
| OSPF | Link State | 현재 대표적인 intra-AS 프로토콜 |
| IS-IS | Link State | OSPF와 유사한 계열 |

## 8-2. OSPF 정의

OSPF는 Open Shortest Path First이다.

- open: 공개 표준이다.
- shortest path first: 최단 경로 우선 방식이다.
- link-state routing 기반이다.
- 각 라우터는 link-state advertisement를 flood한다.
- 모든 라우터는 AS 내부 topology를 알게 된다.
- 각 라우터는 Dijkstra 알고리즘으로 forwarding table을 계산한다.

손필기식 표현:

- “공개 표준, 누구나 구현 가능”
- “링크 상태 라우팅”
- “전체 네트워크 지도를 알고 계산”
- “각 라우터가 자신의 연결 상태를 브로드캐스팅”
- “IP를 직접 사용”
- “OSPF 메시지는 인증되어 보안 유지”

## 8-3. OSPF에서 중요한 개념

### Link-State Advertisement, LSA

- 라우터가 자신의 링크 상태를 알리는 메시지이다.
- 어떤 이웃과 연결되어 있는지,
- 링크 비용이 얼마인지 등을 알린다.

### Flooding

- 정보를 네트워크 전체에 퍼뜨리는 방식이다.
- 한 라우터가 받은 정보를 다른 라우터에게도 전달한다.
- 모든 라우터가 같은 topology 정보를 갖도록 한다.

### Authentication

- OSPF 메시지는 인증을 사용한다.
- 악의적인 라우터가 가짜 라우팅 정보를 넣는 것을 막기 위한 장치이다.

## 8-4. Hierarchical OSPF

OSPF는 큰 AS를 계층적으로 나눌 수 있다.

- local area
- backbone area

왜 나누는가?

- AS가 너무 크면 모든 라우터가 모든 상세 topology를 알기 어렵다.
- 영역별로 정보를 제한하면 확장성이 좋아진다.
- 각 area 내부에서는 자세히 알고,
- 다른 area에 대해서는 요약된 방향 정보만 알 수 있다.

## 8-5. OSPF 라우터 역할

| 역할 | 의미 |
|---|---|
| Internal router | 특정 area 내부에서만 동작하는 라우터 |
| Area border router | area와 backbone을 연결하고 요약 정보를 광고 |
| Backbone router | backbone 영역에서 OSPF를 수행 |
| Boundary router | 다른 AS와 연결되는 라우터 |

손필기 보강:

- local router는 자기 area 안에서 LS 정보를 flood한다.
- area border router는 자기 area의 정보를 요약해 backbone에 전달한다.
- backbone router는 backbone 안에서 라우팅을 수행한다.
- boundary router는 다른 AS와 연결된다.

## 8-6. 시험 포인트

- OSPF는 intra-AS 프로토콜이다.
- OSPF는 link-state 기반이다.
- OSPF는 Dijkstra 알고리즘을 사용한다.
- OSPF는 공개 표준이고 메시지 인증을 지원한다.
- hierarchical OSPF의 local area / backbone 구분을 알아야 한다.

## 8-7. ASCII 와이어프레임

```text
Hierarchical OSPF

        [Backbone Area]
       /       |        \
      /        |         \
 [Area 1]   [Area 2]   [Area 3]

Area 내부:
- 상세 topology 공유

Area 사이:
- area border router가 요약 정보 전달
```

```text
OSPF 흐름

각 라우터가 자신의 링크 상태 파악
        |
        v
LSA 생성
        |
        v
AS 또는 area 내부에 flooding
        |
        v
모든 라우터가 topology DB 구성
        |
        v
Dijkstra로 최단 경로 계산
        |
        v
forwarding table 생성
```

---

# 9. Inter-AS Routing과 BGP (p5-49 ~ p5-63)

## 9-1. BGP 정의

BGP는 Border Gateway Protocol이다.

- inter-domain routing의 사실상 표준이다.
- 서로 다른 AS 사이의 라우팅을 담당한다.
- 인터넷을 연결하는 핵심 프로토콜이다.
- 교안 표현으로는 “glue that holds the Internet together”이다.

손필기식 표현:

- “BGP = 인터 AS 라우팅에 사용”
- “인터넷을 연결하는 풀/접착제”
- “내가 여기 있고, 이런 목적지로 갈 수 있다고 광고”

## 9-2. BGP가 하는 일

BGP는 각 AS에게 다음 수단을 제공한다.

1. eBGP로 이웃 AS에게서 도달 가능 정보를 얻는다.
2. iBGP로 그 정보를 AS 내부 라우터들에게 전파한다.
3. 여러 경로 중 정책에 맞는 경로를 선택한다.
4. 선택한 경로를 다른 AS에 광고할지 결정한다.

## 9-3. eBGP와 iBGP

| 구분 | 의미 | 연결 대상 |
|---|---|---|
| eBGP | external BGP | 서로 다른 AS의 gateway router 사이 |
| iBGP | internal BGP | 같은 AS 내부 라우터 사이 |

손필기식 표현:

- “eBGP: 이웃한 ASes 정보를 받아옴”
- “iBGP: 받아온 정보를 AS 내부에 전파”
- “gateway router는 eBGP와 iBGP를 모두 수행할 수 있음”

## 9-4. BGP session

BGP router들은 peer 관계를 맺고 메시지를 교환한다.

- BGP session은 TCP 연결 위에서 유지된다.
- semi-permanent한 연결이다.
- 경로 광고와 철회 정보를 주고받는다.

## 9-5. BGP는 Path Vector 프로토콜

BGP는 단순히 목적지까지의 거리만 알려주지 않는다.

BGP route는 다음으로 구성된다.

```text
prefix + attributes
```

- prefix: 광고되는 목적지 네트워크 주소 범위
- attributes: 경로 선택에 필요한 속성들

중요한 attributes:

| 속성 | 의미 |
|---|---|
| AS-PATH | 해당 prefix 광고가 지나온 AS들의 목록 |
| NEXT-HOP | 다음 AS로 가기 위해 내부에서 향해야 하는 라우터 |

## 9-6. AS-PATH

AS-PATH는 목적지까지 거쳐야 하는 AS 목록이다.

예:

```text
AS2, AS3, X
```

의미:

- 이 경로를 쓰면 AS2를 거치고,
- 그 다음 AS3을 거쳐,
- 목적지 X에 도달한다.

AS-PATH 역할:

- 경로 길이 비교
- loop 방지
- 정책 판단

## 9-7. NEXT-HOP

NEXT-HOP은 AS 내부 라우터 입장에서 다음으로 보내야 할 gateway를 의미한다.

중요한 점:

- BGP가 “외부 목적지 X는 gateway G를 통해 갈 수 있다”고 알려준다.
- 그러면 AS 내부에서는 OSPF 같은 intra-AS 라우팅으로 G까지 가는 interface를 찾는다.
- 따라서 BGP와 OSPF가 함께 forwarding table에 반영된다.

## 9-8. Policy-based Routing

BGP는 최단 경로만 고르지 않는다.

AS 운영자는 정책을 정할 수 있다.

예:

- 특정 AS를 통과하는 경로는 사용하지 않음
- 고객 트래픽은 전달하지만 경쟁 ISP 사이의 transit은 거부
- 수익이 없는 경로 광고는 하지 않음
- 보안상 특정 경로를 피함

즉, BGP는 성능보다 정책이 우선되는 경우가 많다.

## 9-9. BGP 메시지 종류

| 메시지 | 역할 |
|---|---|
| OPEN | BGP peer와 연결을 열고 인증 |
| UPDATE | 새 경로 광고 또는 기존 경로 철회 |
| KEEPALIVE | 연결 유지, OPEN 확인 |
| NOTIFICATION | 오류 보고 또는 연결 종료 |

## 9-10. Intra-AS와 Inter-AS가 다른 이유

| 기준 | Intra-AS | Inter-AS |
|---|---|---|
| 정책 | 하나의 관리자라 정책 갈등 적음 | AS마다 정책이 다름 |
| 규모 | 한 AS 내부 | 인터넷 전체 규모 |
| 성능 | 성능 최적화 중심 | 정책이 성능보다 우선 가능 |
| 대표 프로토콜 | OSPF | BGP |

## 9-11. Hot Potato Routing

Hot potato routing은 “뜨거운 감자를 오래 들고 있기 싫어서 빨리 넘기는” 방식이다.

의미:

- AS 내부 비용이 가장 낮은 gateway를 선택한다.
- 외부 AS 경로가 전체적으로 더 짧은지는 덜 중요하게 본다.
- 내 AS 입장에서는 빨리 밖으로 보내는 것이 목적이다.

손필기식 표현:

- “빠르게 빠르게”
- “뜨거운 감자를 쥐고 있기 힘드니까”
- “더 많은 AS hop을 거칠 수도 있지만, 내 AS 내부 비용이 작은 gateway를 선택”

## 9-12. BGP route selection 순서

일반적인 선택 기준:

1. local preference 값
2. 짧은 AS-PATH
3. 가장 가까운 NEXT-HOP router
4. 기타 기준

중요:

- 1순위가 local preference라는 점이 중요하다.
- 즉, 정책이 최단 경로보다 먼저 올 수 있다.

## 9-13. 시험 포인트

- BGP는 inter-AS 라우팅 프로토콜이다.
- eBGP와 iBGP를 구분해야 한다.
- BGP route = prefix + attributes이다.
- AS-PATH와 NEXT-HOP은 반드시 알아야 한다.
- BGP는 path vector 프로토콜이다.
- BGP는 정책 기반 라우팅이다.
- hot potato routing은 내부 비용이 가장 작은 gateway를 선택한다.

## 9-14. ASCII 와이어프레임

```text
eBGP / iBGP 감각

        AS1                         AS2
  [router A] --- eBGP --- [router B]
       |                          |
      iBGP                       iBGP
       |                          |
  [router C]                [router D]

다른 AS 사이: eBGP
같은 AS 내부: iBGP
```

```text
BGP route 구조

BGP advertised route
= prefix + attributes

prefix:
  목적지 네트워크 주소 범위

attributes:
  AS-PATH  : 지나온 AS 목록
  NEXT-HOP : 다음 AS로 가기 위한 내부 gateway
```

```text
외부 목적지 X로 가는 table 생성

BGP:
  X는 gateway 1c를 통해 갈 수 있음

OSPF:
  내 위치에서 1c까지는 interface 2가 가장 좋음

Forwarding table:
  destination X -> interface 2
```

```text
Hot potato routing

목적지 X로 가는 후보 gateway

gateway A: 내 AS 내부 비용 2, 외부 AS hop 많음
gateway B: 내 AS 내부 비용 8, 외부 AS hop 적음

Hot potato:
내 AS 비용이 작은 gateway A 선택
```

---

# 10. SDN Control Plane (p5-64 ~ p5-86)

## 10-1. SDN이 등장한 배경

전통적인 인터넷 라우터는 보통 다음을 함께 가진다.

- switching hardware
- routing protocol 구현
- proprietary router OS
- vendor-specific 기능

문제:

- 장비별 설정이 복잡하다.
- 전체 네트워크 정책을 일관되게 적용하기 어렵다.
- traffic engineering이 어렵다.
- 새로운 기능을 빠르게 실험하기 어렵다.

SDN은 이를 바꾸려는 접근이다.

## 10-2. SDN의 핵심 아이디어 4가지

1. Generalized flow-based forwarding
2. Control plane과 data plane 분리
3. Control plane 기능을 data-plane switch 밖으로 이동
4. Programmable control applications

손필기식 표현:

- “스위치는 단순 forwarding에 집중”
- “컨트롤러가 네트워크 전체 상태를 보고 결정”
- “라우팅, 접근 제어, 로드 밸런싱 같은 앱을 위에서 구현”

## 10-3. SDN 구조

SDN 구조는 보통 3층으로 본다.

```text
Network-control applications
        |
        | northbound API
        v
SDN Controller / Network OS
        |
        | southbound API
        v
SDN-controlled switches
```

## 10-4. Data-plane switches

SDN 스위치의 역할:

- 빠르고 단순한 forwarding 수행
- flow table을 기반으로 패킷 처리
- controller가 설치한 규칙을 따른다.
- OpenFlow 같은 API/프로토콜로 제어된다.

## 10-5. SDN Controller

SDN controller는 network operating system처럼 동작한다.

역할:

- 네트워크 상태 정보 유지
- 스위치, 링크, host 정보 관리
- control application에게 abstraction 제공
- southbound API로 스위치 제어
- northbound API로 application과 통신
- 성능, 확장성, 장애 대응을 위해 분산 시스템으로 구현 가능

## 10-6. Network-control applications

network-control app은 실제 제어 기능을 구현한다.

예:

- routing
- access control
- load balancing
- traffic engineering
- firewalling

중요한 점:

- app은 controller가 제공하는 API를 사용한다.
- 장비 vendor와 분리될 수 있다.
- 제어 로직을 더 유연하게 만들 수 있다.

## 10-7. Northbound API와 Southbound API

| API | 방향 | 역할 |
|---|---|---|
| Northbound API | 앱 ↔ 컨트롤러 | 앱이 네트워크 상태를 보고 의도를 표현 |
| Southbound API | 컨트롤러 ↔ 스위치 | 스위치 상태 조회, flow rule 설치 |

손필기식 감각:

- northbound = 위쪽 앱과 말하는 통로
- southbound = 아래쪽 스위치와 말하는 통로

## 10-8. OpenFlow protocol

OpenFlow는 controller와 switch 사이에서 동작한다.

- TCP를 사용해 메시지를 교환한다.
- 선택적으로 암호화 가능하다.
- OpenFlow API와 OpenFlow protocol은 구분해야 한다.

중요:

- API는 어떤 forwarding action을 지정할 수 있는지의 추상화이다.
- protocol은 controller와 switch 사이에서 실제 메시지를 주고받는 방식이다.

## 10-9. OpenFlow 메시지 종류

### Controller-to-switch 메시지

| 메시지 | 역할 |
|---|---|
| features | switch 기능 조회 |
| configure | switch 설정 조회/변경 |
| modify-state | flow entry 추가/삭제/수정 |
| packet-out | 특정 패킷을 특정 포트로 내보내게 함 |

### Switch-to-controller 메시지

| 메시지 | 역할 |
|---|---|
| packet-in | 패킷 처리를 controller에게 넘김 |
| flow-removed | flow table entry 삭제 알림 |
| port status | 포트 상태 변화 알림 |

## 10-10. SDN 상호작용 예시

링크 장애가 발생했다고 하자.

1. switch s1이 링크 장애를 감지한다.
2. s1이 OpenFlow port status 메시지로 controller에게 알린다.
3. controller가 link state 정보를 갱신한다.
4. routing app이 호출된다.
5. routing app이 network graph를 보고 새 경로를 계산한다.
6. flow-table computation component가 새 flow table을 만든다.
7. controller가 OpenFlow로 필요한 switch에 새 table을 설치한다.

## 10-11. ODL과 ONOS

### OpenDaylight, ODL

- SDN controller 구현체 중 하나이다.
- Service Abstraction Layer, SAL을 중심으로 구성된다.
- OpenFlow, NETCONF, SNMP, OVSDB 등 다양한 southbound protocol을 지원한다.
- REST/RESTCONF/NETCONF API를 제공한다.

### ONOS

- SDN controller 구현체 중 하나이다.
- distributed core를 강조한다.
- service reliability, replication, performance scaling을 중시한다.
- intent framework를 제공한다.
- intent는 “어떻게 할지”보다 “무엇을 원하는지”를 표현하는 고수준 방식이다.

## 10-12. SDN의 도전 과제

SDN은 강력하지만 다음 과제가 있다.

- controller의 신뢰성
- 장애에 대한 견고성
- 보안
- 대규모 확장성
- 단일 AS를 넘어 인터넷 규모로 확장하는 문제
- 실시간/초신뢰/초보안 네트워크 요구사항 대응

## 10-13. 시험 포인트

- SDN은 logically centralized control이다.
- control plane과 data plane을 분리한다.
- controller는 network OS처럼 동작한다.
- northbound API와 southbound API를 구분한다.
- OpenFlow는 controller-switch 사이 프로토콜이다.
- packet-in과 packet-out 방향을 헷갈리지 않는다.
- SDN은 traffic engineering을 더 유연하게 만든다.

## 10-14. ASCII 와이어프레임

```text
SDN 기본 구조

+----------------------------------+
| Network-control Applications     |
| routing / ACL / load balancing   |
+----------------------------------+
                 |
                 | Northbound API
                 v
+----------------------------------+
| SDN Controller / Network OS      |
| topology / state / flow rules    |
+----------------------------------+
                 |
                 | Southbound API
                 v
+----------------------------------+
| SDN Switches                     |
| flow table 기반 forwarding       |
+----------------------------------+
```

```text
링크 장애 시 SDN 동작

Link failure 발생
      |
      v
Switch -> Controller
(port status)
      |
      v
Controller 상태 DB 갱신
      |
      v
Routing app 호출
      |
      v
새 경로 계산
      |
      v
새 flow table 계산
      |
      v
Controller -> Switch
(modify-state)
```

```text
OpenFlow 방향 감각

Controller -> Switch
  features
  configure
  modify-state
  packet-out

Switch -> Controller
  packet-in
  flow-removed
  port status
```

---

# 11. ICMP와 Traceroute (p5-87 ~ p5-90)

## 11-1. ICMP 정의

ICMP는 Internet Control Message Protocol이다.

- host와 router가 네트워크 계층 정보를 주고받는 데 사용한다.
- 오류 보고에 사용된다.
- 진단 도구에 사용된다.
- ICMP 메시지는 IP datagram 안에 담긴다.

## 11-2. ICMP의 대표 용도

1. 오류 보고
   - 목적지 네트워크 도달 불가
   - 목적지 호스트 도달 불가
   - 목적지 포트 도달 불가
   - TTL 만료
   - 잘못된 IP header

2. 진단
   - ping
   - traceroute

## 11-3. ICMP message 구조 감각

ICMP 메시지는 보통 다음을 가진다.

- type
- code
- 오류를 유발한 IP datagram의 일부 정보

대표 type/code:

| Type | Code | 의미 |
|---|---|---|
| 0 | 0 | echo reply |
| 8 | 0 | echo request |
| 3 | 3 | destination port unreachable |
| 11 | 0 | TTL expired |
| 12 | 0 | bad IP header |

## 11-4. ping

ping은 ICMP echo request와 echo reply를 사용한다.

흐름:

```text
Host A -> Host B : ICMP echo request
Host B -> Host A : ICMP echo reply
```

이를 통해 다음을 확인할 수 있다.

- 상대 호스트가 응답하는지
- 왕복 시간 RTT가 어느 정도인지
- 중간 네트워크 연결이 정상인지

## 11-5. traceroute

traceroute는 목적지까지 가는 경로에 있는 router들을 알아내는 도구이다.

핵심 아이디어:

- TTL을 1부터 증가시키며 UDP segment를 보낸다.
- TTL이 0이 된 router는 datagram을 버린다.
- 그 router는 source에게 ICMP TTL expired 메시지를 보낸다.
- source는 이 응답을 보고 몇 번째 router인지 기록한다.

## 11-6. traceroute 종료 조건

마지막 목적지 host에 UDP segment가 도착한다.

그런데 traceroute는 보통 사용하지 않는 port로 보낸다.

목적지는 다음 메시지를 보낸다.

```text
ICMP destination port unreachable
Type 3, Code 3
```

source는 이 메시지를 받으면 목적지에 도달했다고 판단하고 멈춘다.

## 11-7. 시험 포인트

- ICMP는 IP 위에 실리지만 네트워크 계층 제어 메시지이다.
- ping은 echo request/reply이다.
- traceroute는 TTL과 ICMP TTL expired를 이용한다.
- 최종 목적지에서는 port unreachable이 돌아와 종료된다.
- TTL expired는 type 11 code 0이다.
- port unreachable은 type 3 code 3이다.

## 11-8. ASCII 와이어프레임

```text
ping

A --------------------> B
   ICMP echo request

A <-------------------- B
   ICMP echo reply
```

```text
traceroute

TTL = 1
Source -> R1 -> TTL 0
R1 -> Source : ICMP TTL expired

TTL = 2
Source -> R1 -> R2 -> TTL 0
R2 -> Source : ICMP TTL expired

TTL = 3
Source -> R1 -> R2 -> R3 -> TTL 0
R3 -> Source : ICMP TTL expired

...

Destination 도착
Destination -> Source : ICMP port unreachable
=> traceroute 종료
```

---

# 12. Network Management, SNMP, NETCONF/YANG (p5-91 ~ p5-102)

## 12-1. Network Management란?

Network management는 네트워크 장비와 서비스를 운영하기 위한 관리 활동이다.

포함되는 일:

- 장비 배치
- 장비 설정
- 상태 모니터링
- 장애 감지
- 성능 분석
- 테스트
- 제어
- 운영 정책 반영

감각적으로는 다음과 같다.

- 라우터와 스위치를 그냥 설치만 하는 것이 아니다.
- 상태를 보고,
- 설정을 바꾸고,
- 문제가 생기면 감지하고,
- 전체 품질을 유지하는 작업이다.

## 12-2. Network Management 구성 요소

| 구성 요소 | 의미 |
|---|---|
| Managing server/controller | 관리자 또는 관리 애플리케이션이 사용하는 서버 |
| Managed device | 관리 대상 장비, 예: router, switch |
| Agent | 장비 내부에서 관리 정보를 제공하는 소프트웨어 |
| Data | 설정, 상태, 통계 정보 |
| Management protocol | 서버와 장비가 관리 정보를 주고받는 규칙 |

## 12-3. 관리 방식 3가지

### CLI

- 사람이 직접 장비에 접속해 명령 입력
- 예: SSH로 접속해서 설정 명령 실행
- 장비별 수동 관리에 가깝다.

### SNMP/MIB

- SNMP로 장비의 MIB 정보를 조회하거나 설정한다.
- 전통적인 모니터링 방식에 많이 사용된다.

### NETCONF/YANG

- 더 구조적이고 추상적인 관리 방식이다.
- 여러 장비의 설정을 일관되게 관리하는 데 초점이 있다.
- YANG은 데이터 모델링 언어이다.
- NETCONF는 YANG 기반 데이터와 명령을 주고받는 프로토콜이다.

## 12-4. SNMP 정의

SNMP는 Simple Network Management Protocol이다.

SNMP의 주요 구조:

- manager
- agent
- managed device
- MIB

SNMP는 두 방식으로 정보를 전달한다.

1. request/response mode
2. trap mode

## 12-5. SNMP Request/Response Mode

관리 서버가 장비에게 묻는다.

```text
Manager -> Agent : 이 값 알려줘
Agent -> Manager : 값은 이것입니다
```

예:

- 현재 interface 상태 알려줘
- UDP datagram 수 알려줘
- 특정 MIB 값 설정해줘

## 12-6. SNMP Trap Mode

장비가 예외 상황을 먼저 알린다.

```text
Agent -> Manager : 문제가 발생했습니다
```

예:

- 포트 다운
- 장비 이상
- 임계치 초과

## 12-7. SNMP 메시지 종류

| 메시지 | 방향 | 기능 |
|---|---|---|
| GetRequest | manager -> agent | 특정 데이터 요청 |
| GetNextRequest | manager -> agent | 다음 데이터 요청 |
| GetBulkRequest | manager -> agent | 여러 데이터 블록 요청 |
| SetRequest | manager -> agent | MIB 값 설정 |
| Response | agent -> manager | 요청에 대한 응답 |
| Trap | agent -> manager | 예외 이벤트 알림 |

## 12-8. MIB와 SMI

### MIB

MIB는 Management Information Base이다.

- 관리 대상 데이터의 모음이다.
- 장비의 상태, 통계, 일부 설정 정보를 담는다.
- 각 항목은 Object ID, 이름, 타입 등을 가진다.

예:

- UDPInDatagrams
- UDPNoPorts
- UDPOutDatagrams

### SMI

SMI는 Structure of Management Information이다.

- MIB 데이터를 정의하는 언어/규칙이다.
- 어떤 타입으로 표현할지 정한다.

## 12-9. NETCONF 정의

NETCONF는 네트워크 장비 설정을 더 적극적으로 관리하기 위한 프로토콜이다.

특징:

- managing server와 managed device 사이에서 동작한다.
- 설정 조회, 수정, 활성화가 가능하다.
- operational data와 statistics 조회 가능하다.
- notification 구독이 가능하다.
- RPC 패러다임을 사용한다.
- 메시지는 XML로 인코딩된다.
- TLS 같은 secure reliable transport 위에서 동작할 수 있다.

## 12-10. NETCONF 흐름

```text
Session 시작
  <hello> 교환

요청/응답 반복
  <rpc>
  <rpc-reply>

이벤트 발생 시
  <notification>

Session 종료
  <close-session>
```

## 12-11. NETCONF 주요 operation

| Operation | 의미 |
|---|---|
| <get-config> | 설정 정보 조회 |
| <get> | 설정 + 운영 상태 조회 |
| <edit-config> | 설정 변경 |
| <lock> | 설정 저장소 잠금 |
| <unlock> | 설정 저장소 잠금 해제 |
| <create-subscription> | 이벤트 알림 구독 |
| <notification> | 이벤트 알림 |

## 12-12. Atomic commit 감각

NETCONF는 여러 장비 설정을 다룰 때 일관성이 중요하다.

atomic commit은 다음 의미이다.

- 여러 설정 변경을 하나의 작업처럼 처리한다.
- 전부 성공하면 반영한다.
- 중간에 실패하면 rollback한다.

감각:

```text
전부 성공 -> commit
하나라도 실패 -> rollback
```

## 12-13. YANG 정의

YANG은 NETCONF에서 사용할 관리 데이터를 정의하는 데이터 모델링 언어이다.

YANG이 정의하는 것:

- 데이터 구조
- 문법
- 의미
- 제약 조건
- 유효한 설정 조건

YANG의 장점:

- 장비 설정을 구조화할 수 있다.
- 잘못된 설정을 미리 막을 수 있다.
- NETCONF XML 문서를 생성할 수 있다.
- 여러 장비 설정의 일관성을 높일 수 있다.

## 12-14. SNMP vs NETCONF/YANG

| 구분 | SNMP/MIB | NETCONF/YANG |
|---|---|---|
| 중심 | 상태 조회, 모니터링 | 설정 관리, 구조적 제어 |
| 데이터 모델 | MIB/SMI | YANG |
| 메시지 스타일 | get/set/trap | RPC/XML |
| 장점 | 단순하고 오래 사용됨 | 일관된 설정 관리에 강함 |
| 감각 | “장비 상태 좀 알려줘” | “여러 장비 설정을 모델 기반으로 관리하자” |

## 12-15. 시험 포인트

- network management 구성 요소를 알아야 한다.
- SNMP의 manager/agent/MIB 구조를 설명할 수 있어야 한다.
- request/response와 trap 차이를 구분한다.
- MIB는 관리 정보 데이터베이스이다.
- NETCONF는 설정 관리에 강한 프로토콜이다.
- YANG은 데이터 모델링 언어이다.
- NETCONF는 RPC/XML 기반 흐름을 가진다.

## 12-16. ASCII 와이어프레임

```text
SNMP request/response

[Managing Server] ---- GetRequest ----> [Agent / Device]
[Managing Server] <---- Response ------ [Agent / Device]
```

```text
SNMP trap

[Managing Server] <---- Trap ---------- [Agent / Device]

장비가 먼저 예외 상황을 알림
```

```text
NETCONF/YANG 관계

YANG
  - 데이터 구조 정의
  - 제약 조건 정의
        |
        v
NETCONF XML 메시지
  - <rpc>
  - <edit-config>
  - <rpc-reply>
        |
        v
Managed Device 설정 변경
```

```text
Atomic commit

설정 변경 A 성공
설정 변경 B 성공
설정 변경 C 실패
       |
       v
전체 rollback

=> 일부만 바뀌는 위험 방지
```

---

# 13. 생소한 개념 미니 사전

## 13-1. Prefix

- 네트워크 주소 범위이다.
- BGP에서 광고되는 목적지 단위이다.
- 예: 어떤 AS가 “이 prefix로 오는 트래픽은 내가 처리 가능”이라고 알린다.

## 13-2. AS-PATH

- 목적지 prefix까지 가기 위해 거치는 AS 목록이다.
- BGP 경로 선택과 loop 방지에 사용된다.

## 13-3. NEXT-HOP

- 외부 목적지로 가기 위해 내부에서 먼저 향해야 하는 gateway router이다.
- BGP 정보와 intra-AS 라우팅 정보를 연결하는 핵심이다.

## 13-4. Policy-based Routing

- 단순히 가장 짧은 경로가 아니라 운영 정책에 따라 경로를 선택하는 방식이다.
- BGP에서 매우 중요하다.

## 13-5. Hot Potato Routing

- 내 AS 내부 비용이 가장 낮은 출구를 선택하는 방식이다.
- 전체 인터넷 경로가 최단이 아닐 수도 있다.

## 13-6. Count-to-infinity

- DV에서 나쁜 소식이 늦게 퍼지는 문제이다.
- 라우터들이 서로 잘못된 경로 정보를 믿고 비용을 조금씩 올리는 현상이다.

## 13-7. Black-holing

- 잘못된 라우터가 낮은 비용을 광고하여 트래픽을 끌어들이는 문제이다.
- 실제 전달이 안 되면 패킷이 사라지는 것처럼 된다.

## 13-8. Northbound / Southbound API

- northbound API는 SDN controller와 app 사이의 인터페이스이다.
- southbound API는 controller와 switch 사이의 인터페이스이다.

## 13-9. Flow Table

- SDN switch가 패킷을 어떻게 처리할지 적어둔 규칙 표이다.
- match 조건과 action으로 구성된다.

## 13-10. MIB

- SNMP에서 관리 대상 정보를 모아둔 데이터베이스이다.
- Object ID로 관리 항목을 구분한다.

## 13-11. YANG

- NETCONF에서 사용할 관리 데이터의 구조와 제약을 정의하는 언어이다.
- 설정 오류를 줄이고 자동화에 유리하다.

---

# 마지막. 헷갈리는 비교 정리

## 1. Forwarding vs Routing

| 구분 | Forwarding | Routing |
|---|---|---|
| 평면 | Data Plane | Control Plane |
| 역할 | 입력 포트에서 출력 포트로 패킷 이동 | 출발지-목적지 경로 결정 |
| 속도 | 매우 빠름 | 상대적으로 느려도 됨 |
| 기준 | forwarding table | routing algorithm/protocol |

## 2. Per-router Control vs SDN Control

| 구분 | Per-router | SDN |
|---|---|---|
| 제어 위치 | 각 라우터 내부 | 원격 컨트롤러 |
| 계산 방식 | 분산 라우팅 알고리즘 | 중앙/논리적 중앙 계산 |
| 장점 | 전통적이고 분산적 | 관리, 정책, 자동화 유리 |
| 단점 | 전체 제어 어려움 | 컨트롤러 신뢰성 중요 |

## 3. Link State vs Distance Vector

| 구분 | Link State | Distance Vector |
|---|---|---|
| 대표 | Dijkstra | Bellman-Ford |
| 정보 | 전체 topology | 이웃의 거리 정보 |
| 계산 | 전체 지도 기반 | 이웃 정보 반복 갱신 |
| 문제 | oscillation | count-to-infinity |

## 4. OSPF vs BGP

| 구분 | OSPF | BGP |
|---|---|---|
| 범위 | Intra-AS | Inter-AS |
| 기반 | Link State | Path Vector |
| 중점 | 성능/최단 경로 | 정책/도달 가능성 |
| 사용 위치 | AS 내부 | AS 사이 |

## 5. eBGP vs iBGP

| 구분 | eBGP | iBGP |
|---|---|---|
| 의미 | external BGP | internal BGP |
| 연결 | 서로 다른 AS | 같은 AS 내부 |
| 역할 | 외부 AS에서 경로 학습 | AS 내부로 경로 정보 전파 |

## 6. SNMP vs NETCONF/YANG

| 구분 | SNMP | NETCONF/YANG |
|---|---|---|
| 주요 용도 | 모니터링, 상태 조회 | 설정 관리, 자동화 |
| 데이터 모델 | MIB/SMI | YANG |
| 메시지 | get/set/trap | RPC/XML |
| 감각 | 장비 상태 관리 | 네트워크 설정 관리 |

---

# 마지막 체크리스트

## 개념 체크

- [ ] data plane과 control plane을 구분할 수 있다.
- [ ] forwarding과 routing 차이를 설명할 수 있다.
- [ ] per-router control과 SDN control을 비교할 수 있다.
- [ ] path와 cost의 의미를 설명할 수 있다.
- [ ] graph G=(N,E)의 의미를 안다.
- [ ] Link State와 Distance Vector를 구분할 수 있다.
- [ ] Dijkstra 알고리즘의 D(v), p(v), N'을 설명할 수 있다.
- [ ] Bellman-Ford 식을 말로 풀 수 있다.
- [ ] count-to-infinity를 예시로 설명할 수 있다.
- [ ] route oscillation이 왜 생기는지 안다.

## 프로토콜 체크

- [ ] OSPF가 intra-AS 프로토콜임을 안다.
- [ ] BGP가 inter-AS 프로토콜임을 안다.
- [ ] eBGP와 iBGP를 구분할 수 있다.
- [ ] AS-PATH와 NEXT-HOP의 의미를 안다.
- [ ] hot potato routing을 설명할 수 있다.
- [ ] SDN의 northbound/southbound API를 구분할 수 있다.
- [ ] OpenFlow packet-in/packet-out 방향을 구분할 수 있다.
- [ ] ICMP와 ping/traceroute 관계를 안다.
- [ ] SNMP의 request/response와 trap을 구분할 수 있다.
- [ ] NETCONF와 YANG의 역할을 구분할 수 있다.

## 시험 직전 핵심 암기

```text
Forwarding = data plane = 표 보고 패킷 내보내기
Routing    = control plane = 경로와 표 만들기

LS = 전체 지도 + Dijkstra
DV = 이웃 정보 + Bellman-Ford

OSPF = AS 내부, link-state
BGP  = AS 사이, path-vector, policy 중심

SDN = control/data 분리 + controller 중심
ICMP = 오류 보고 + ping/traceroute
SNMP = 장비 상태 관리
NETCONF/YANG = 구조적 설정 관리
```

---

# 전사용 압축 버전

- 컨트롤 플레인은 라우팅을 담당한다.
- 데이터 플레인은 포워딩을 담당한다.
- 포워딩은 이미 만들어진 forwarding table을 보고 패킷을 내보내는 일이다.
- 라우팅은 목적지까지 어떤 경로를 쓸지 결정하고 forwarding table을 만드는 일이다.
- 전통적 방식은 각 라우터가 직접 라우팅 알고리즘을 수행한다.
- SDN 방식은 원격 컨트롤러가 경로를 계산하고 스위치에 규칙을 설치한다.
- 라우팅 프로토콜의 목표는 좋은 path를 찾는 것이다.
- path는 패킷이 지나가는 라우터들의 순서이다.
- cost는 경로 선택을 위한 비용 값이며 낮을수록 좋다.
- Link State는 모든 라우터가 전체 topology와 link cost를 알고 계산한다.
- Link State의 대표 알고리즘은 Dijkstra이다.
- Dijkstra는 출발지에서 모든 목적지까지의 최소 비용 경로를 찾는다.
- Distance Vector는 각 라우터가 이웃의 거리 정보를 받아 자기 정보를 갱신한다.
- Distance Vector의 대표 식은 `D_x(y)=min_v{c_x,v+D_v(y)}`이다.
- DV에서 좋은 소식은 빨리 퍼지지만 나쁜 소식은 느리게 퍼질 수 있다.
- count-to-infinity는 라우터들이 서로 잘못된 경로 정보를 믿고 비용을 조금씩 올리는 문제이다.
- 실제 인터넷은 너무 커서 하나의 평평한 네트워크로 라우팅할 수 없다.
- 그래서 AS 단위로 나누어 관리한다.
- intra-AS는 AS 내부 라우팅이고 inter-AS는 AS 사이 라우팅이다.
- OSPF는 대표적인 intra-AS link-state 프로토콜이다.
- BGP는 대표적인 inter-AS path-vector 프로토콜이다.
- BGP route는 prefix와 attributes로 구성된다.
- AS-PATH는 지나온 AS 목록이고 NEXT-HOP은 다음으로 향해야 할 gateway이다.
- BGP는 최단 경로보다 정책이 더 중요할 수 있다.
- Hot potato routing은 내 AS 내부 비용이 가장 낮은 출구로 빨리 내보내는 방식이다.
- SDN은 control plane과 data plane을 분리한다.
- SDN controller는 네트워크 상태를 보고 flow table을 계산한다.
- OpenFlow는 controller와 switch 사이에서 메시지를 주고받는 프로토콜이다.
- ICMP는 네트워크 계층 오류 보고와 진단을 위한 프로토콜이다.
- ping은 ICMP echo request/reply를 사용한다.
- traceroute는 TTL expired와 port unreachable 메시지를 이용한다.
- SNMP는 manager가 agent를 통해 장비 상태를 조회하거나 trap을 받는 방식이다.
- NETCONF는 장비 설정을 RPC/XML 기반으로 관리한다.
- YANG은 NETCONF 설정 데이터의 구조와 제약을 정의하는 모델링 언어이다.


# 네트워크 Chapter 3 손필기 전사용 노트
부제: Transport Layer를 손으로 다시 옮기기 쉽게 재구성한 버전  
기준: Chapter 3 원본 슬라이드 + 사용자의 TCP 손필기 메모 반영

---

## 이 장에서 먼저 잡아야 할 핵심 15문장

1. **Transport Layer는 host 대 host가 아니라 process 대 process의 논리적 통신을 제공한다.**
2. **네트워크 계층은 집까지 보내고, 전송 계층은 집 안에서 누구에게 줄지 구분한다.**
3. **이 구분의 핵심이 multiplexing / demultiplexing이다.**
4. **UDP는 연결 설정이 없고 빠르지만, 신뢰성·순서 보장·혼잡 제어를 기본 제공하지 않는다.**
5. **TCP는 reliable, in-order, flow control, congestion control, connection setup을 제공한다.**
6. **Reliable Data Transfer의 핵심은 오류/손실이 있는 채널 위에 신뢰성을 덧씌우는 것이다.**
7. **Stop-and-Wait는 이해는 쉽지만 성능이 매우 비효율적이다.**
8. **그래서 pipelining이 필요하고, 대표 방식이 Go-Back-N과 Selective Repeat이다.**
9. **TCP의 sequence number는 세그먼트 번호가 아니라 byte-stream 기준 번호다.**
10. **ACK는 다음에 받고 싶은 바이트 번호를 의미하며, cumulative ACK이다.**
11. **Flow Control은 receiver 보호, Congestion Control은 network 보호다.**
12. **2-way handshake는 half-open connection 문제 때문에 위험하고, TCP는 3-way handshake를 쓴다.**
13. **RTT는 변하므로 SampleRTT 하나만 보지 않고 EstimatedRTT와 DevRTT로 timeout을 잡는다.**
14. **TCP 혼잡 제어의 큰 그림은 AIMD + Slow Start + ssthresh이다.**
15. **최근 흐름에서는 QUIC/HTTP3처럼 UDP 위에 신뢰성·보안·혼잡 제어를 올리는 방향도 중요하다.**

---

## 0. 큰 그림: Chapter 2와 Chapter 3의 연결

- Chapter 2가 **응용 계층에서 어떤 앱이 어떤 서비스를 원하는가**를 다뤘다면,
- Chapter 3는 **그 서비스를 전송 계층이 실제로 어떻게 제공하는가**를 다룬다.

즉 흐름은 다음이다.

```text
앱이 원하는 것
-> 신뢰성? 속도? 지연? 보안?
-> TCP를 쓸까 UDP를 쓸까
-> 전송 계층 내부에서 어떻게 그 요구를 만족시킬까
```

---

## 1. Transport Layer의 본질

### 1-1. 역할
- 서로 다른 호스트에서 실행되는 **응용 프로세스들 사이의 논리적 통신** 제공
- 송신 측:
  - 앱 메시지를 받음
  - 세그먼트로 나눔
  - 헤더 값을 붙임
  - 네트워크 계층(IP)으로 넘김
- 수신 측:
  - IP로부터 세그먼트 받음
  - 헤더 확인
  - 원래 앱 메시지로 재조립
  - 올바른 소켓에 전달

### 1-2. 네트워크 계층과 차이
- **Network layer**: host to host
- **Transport layer**: process to process

### 1-3. 비유
- 네트워크 계층 = 집 주소까지 우편 배달
- 전송 계층 = 집 안 여러 사람 중 누구에게 줄지 구분

---

## 2. Multiplexing / Demultiplexing

### 2-1. 왜 필요한가?
한 호스트 안에는 여러 프로세스가 동시에 돌 수 있다.  
따라서 "이 세그먼트가 어느 프로세스용인가?"를 구분해야 한다.

### 2-2. 핵심 기준
- IP 주소
- Port 번호

### 2-3. UDP와 TCP의 demux 기준 차이
#### UDP
- **destination port 번호만 보고** 같은 소켓으로 보낼 수 있음
- 같은 목적지 포트라면 출발지 IP/포트가 달라도 같은 소켓에 들어올 수 있음

#### TCP
- **4-tuple**로 식별
```text
(source IP, source port, dest IP, dest port)
```
- 서버는 같은 dest port(예: 80)를 써도, 서로 다른 클라이언트와의 연결을 각각 다른 소켓으로 구분 가능

### 2-4. 시험 포인트
- **UDP = 목적지 포트 중심**
- **TCP = 4개 값 전체로 연결 식별**

---

## 3. UDP

### 3-1. 성격
- connectionless
- best effort
- 신뢰성 없음
- 순서 보장 없음
- congestion control 없음
- handshake 없음
- 헤더가 작고 단순함

### 3-2. 왜 쓰는가?
- 연결 설정 RTT가 없다
- 빠르고 단순하다
- 약간의 손실을 허용하는 앱에 적합하다
- 필요하면 앱 레벨에서 신뢰성/혼잡 제어를 따로 구현할 수 있다

### 3-3. 자주 나오는 예
- DNS
- SNMP
- 실시간 스트리밍
- HTTP/3의 기반(UDP 위에 QUIC)

### 3-4. UDP 헤더
- source port
- destination port
- length
- checksum

### 3-5. UDP checksum
목적:
- 전송 중 비트 오류 검출

주의:
- 오류를 **완벽히 막는 것**이 아니라 **검출하려는 것**
- checksum이 같다고 해서 반드시 오류가 없다는 뜻은 아님  
  → 보호가 약할 수 있음

### 3-6. 한 줄 요약
**UDP는 "최소 기능만 제공하는 전송 계층"이다.**

---

## 4. Reliable Data Transfer(RDT)의 본질

### 4-1. 문제의 본질
현실 채널은
- 비트가 깨질 수 있고
- 패킷이 손실될 수 있고
- 순서가 바뀔 수 있다

그런데 앱은 "정확히 도착했다"는 신뢰성을 원한다.  
그래서 **불안정한 채널 위에 프로토콜로 신뢰성을 덧씌우는 것**이 RDT의 핵심이다.

### 4-2. 핵심 아이디어
- checksum
- ACK / NAK
- sequence number
- retransmission
- timer

---

## 5. rdt1.0 ~ rdt3.0 흐름 이해

## 5-1. rdt1.0
- 채널이 완벽히 신뢰적이라고 가정
- 그냥 보내고 받으면 끝
- 현실성 거의 없음

## 5-2. rdt2.0
- 비트 오류 가능
- checksum 사용
- receiver가 ACK/NAK 보냄
- sender는 NAK 받으면 재전송

문제:
- **ACK/NAK 자체가 손상되면?**
- sender가 receiver 상태를 정확히 모름

## 5-3. rdt2.1
- sequence number 추가
- duplicate packet 구분 가능
- ACK/NAK 깨짐에도 재전송 + 중복 판별 가능

## 5-4. rdt2.2
- NAK 없이 ACK만으로 처리
- 중복 ACK를 사실상 NAK처럼 사용
- TCP가 이 철학과 비슷하게 감

## 5-5. rdt3.0
- 이제 비트 오류뿐 아니라 **패킷 손실**도 고려
- ACK가 안 오면 timeout 후 재전송
- 즉,
```text
오류 -> checksum
중복 -> sequence number
손실 -> timer + retransmission
```

### 5-6. 손필기 포인트
```text
rdt 진화 순서:
오류 없음 -> 비트 오류 -> ACK/NAK 오류 -> 패킷 손실
```

---

## 6. Stop-and-Wait와 왜 느린가

### 6-1. 방식
- 한 패킷 보내고
- ACK 올 때까지 기다리고
- 다음 패킷 보냄

### 6-2. 문제
링크가 빨라도 RTT 동안 기다리는 시간이 커서 비효율적이다.

### 6-3. 핵심 문장
**채널은 빠른데 프로토콜이 기다리느라 성능을 깎아먹는다.**

---

## 7. Pipelining

### 7-1. 왜 필요한가?
Stop-and-Wait는 너무 비효율적이므로  
**ACK를 기다리지 않고 여러 패킷을 한꺼번에 띄워둔다.**

### 7-2. 필요해지는 것
- 더 큰 sequence number 범위
- sender/receiver 버퍼링
- window 개념

---

## 8. Go-Back-N (GBN)

### 8-1. sender
- 최대 N개까지 ACK 안 받은 패킷을 보낼 수 있음
- cumulative ACK 사용
- 가장 오래된 unACKed packet 하나에 대한 타이머 사용
- timeout 나면 **그 패킷부터 뒤의 것까지 전부 재전송**

### 8-2. receiver
- in-order만 정상 수용
- out-of-order는 보통 버리거나 구현에 따라 처리
- 항상 지금까지 연속적으로 잘 받은 마지막 패킷 기준 ACK

### 8-3. 장단점
장점:
- 구현이 단순

단점:
- 하나만 빠져도 뒤에 이미 받은 것까지 다시 보내게 되어 비효율

### 8-4. 기억 문장
**GBN = 하나 빠지면 뒤도 같이 다시 간다.**

---

## 9. Selective Repeat (SR)

### 9-1. sender
- 각 패킷별로 개별 ACK 관리
- 개별 timeout / 개별 재전송

### 9-2. receiver
- out-of-order도 버퍼링 가능
- 나중에 빠진 것 오면 순서 맞춰 전달

### 9-3. 장단점
장점:
- 필요한 패킷만 재전송해서 효율적

단점:
- 구현 복잡
- window와 sequence number 관계를 조심해야 함

### 9-4. 시험 포인트
**SR은 효율적이지만 헷갈리기 쉽다.**  
특히 sequence number space가 window보다 충분히 커야 한다.

### 9-5. 기억 문장
**SR = 필요한 것만 골라서 다시 보낸다.**

---

## 10. TCP 개요

### 10-1. TCP가 제공하는 것
- reliable delivery
- in-order delivery
- cumulative ACK
- pipelining
- flow control
- congestion control
- connection-oriented
- full duplex
- point-to-point

### 10-2. 중요한 해석
TCP는 단순히 "안전하다"가 아니라,
**신뢰성 + 수신측 보호 + 네트워크 보호 + 연결 관리**를 모두 포함한 종합 패키지다.

---

## 11. TCP segment structure

### 11-1. 자주 보는 필드
- source port
- destination port
- sequence number
- acknowledgement number
- receive window(rwnd)
- checksum
- SYN / ACK / FIN
- options
- data

### 11-2. 특히 중요
#### sequence number
- 세그먼트 번호가 아니라
- **이 세그먼트 데이터의 첫 바이트 번호**

#### acknowledgement number
- **다음에 받고 싶은 바이트 번호**
- cumulative ACK

#### rwnd
- receiver가 현재 더 받아줄 수 있는 여유 공간

---

## 12. TCP sequence number와 ACK

### 12-1. sequence number
바이트 스트림 기준으로 센다.  
즉 세그먼트 수를 세는 것이 아니다.

### 12-2. ACK
"여기까지 잘 받았고, 이제 다음 번호부터 보내라"는 뜻이다.

예:
- ACK=43이면
- 42번 바이트까지는 받았고
- 43번 바이트부터 기대한다는 뜻

### 12-3. cumulative ACK
중간 ACK 하나가 앞선 여러 데이터의 수신 사실까지 한 번에 포함할 수 있다.

### 12-4. 손필기 포인트
```text
Seq = 내가 보내는 데이터의 시작 바이트 번호
ACK = 네가 다음에 보내야 할 바이트 번호
```

---

## 13. TCP RTT, EstimatedRTT, Timeout

### 13-1. RTT
- Round Trip Time
- 세그먼트가 갔다가 ACK가 돌아오는 왕복 시간
- 고정이 아니라 계속 변한다

### 13-2. 왜 SampleRTT 하나만 보면 안 되는가?
- 현재 네트워크 상황에 따라 튄다
- 순간값 하나만 믿으면 timeout이 너무 짧거나 길어질 수 있다

### 13-3. EstimatedRTT
```text
EstimatedRTT = (1 - α) * EstimatedRTT + α * SampleRTT
```
- 지수 가중 이동 평균(EWMA)
- 보통 α = 0.125
- 과거 흐름을 더 중시하면서 현재 값을 일부 반영

### 13-4. DevRTT
```text
DevRTT = (1 - β) * DevRTT + β * |SampleRTT - EstimatedRTT|
```
- RTT 변동 폭 반영
- 보통 β = 0.25

### 13-5. TimeoutInterval
```text
TimeoutInterval = EstimatedRTT + 4 * DevRTT
```
- 평균 RTT + 안전 마진
- 변동이 크면 timeout도 더 넉넉하게 잡음

### 13-6. 손필기 해석
- SampleRTT = 순간값
- EstimatedRTT = 부드러운 평균 추세
- DevRTT = 얼마나 흔들리는지
- Timeout = 평균 + 여유

### 13-7. 외우기 좋은 문장
**TCP는 고정 timeout이 아니라, 네트워크 상태를 보며 timeout을 동적으로 조정한다.**

---

## 14. TCP sender / receiver 동작

### 14-1. sender 쪽 핵심
- 앱에서 데이터 오면 세그먼트 생성
- 아직 타이머가 없으면 시작
- ACK 오면 ACK된 범위 갱신
- timeout 나면 재전송 후 타이머 재시작

### 14-2. receiver ACK 생성 규칙
- in-order 세그먼트 1개 도착 → 잠깐 delayed ACK 가능
- 연속된 in-order 세그먼트 2개 정도 오면 누적 ACK 즉시 전송
- out-of-order 오면 duplicate ACK 즉시 전송
- gap을 메우는 세그먼트 오면 ACK 전송

### 14-3. 손필기 포인트
- 타이머는 단순화하면 **가장 오래된 미확인 세그먼트 기준**
- ACK는 cumulative
- duplicate ACK는 이상 신호

---

## 15. TCP 재전송 시나리오와 Fast Retransmit

### 15-1. ACK loss
- 데이터는 잘 도착했는데 ACK가 유실될 수 있음
- sender는 timeout 후 재전송할 수 있음
- receiver는 중복 데이터임을 인지해야 함

### 15-2. premature timeout
- 사실 네트워크가 느려서 ACK가 늦었을 뿐인데
- sender가 성급하게 timeout을 내고 재전송할 수 있음

### 15-3. cumulative ACK의 장점
- 이전 ACK가 유실돼도
- 더 큰 ACK가 오면 앞부분까지 한 번에 커버 가능

### 15-4. fast retransmit
- timeout까지 기다리면 느리다
- **같은 ACK가 3번 더 반복되면(triple duplicate ACK)**
  누락 세그먼트가 있다고 보고 빨리 재전송

### 15-5. 핵심 문장
**duplicate ACK 3개 = "중간 하나가 비었을 가능성이 높다"는 강한 신호**

---

## 16. TCP Flow Control

### 16-1. 왜 필요한가?
receiver가 앱에서 데이터를 꺼내는 속도보다 sender가 더 빨리 보내면  
receiver buffer overflow가 발생할 수 있다.

### 16-2. 핵심 개념
- **rwnd = receive window**
- receiver가 "지금 이만큼 더 받을 수 있어"라고 sender에게 광고(advertise)함

### 16-3. sender 동작
- ACK 안 된 in-flight 데이터 총량을 rwnd 이하로 유지하려고 함

### 16-4. 의미
Flow Control은
- 라우터 보호가 아니라
- **수신측 버퍼 보호**이다.

### 16-5. 손필기식 표현
```text
Flow Control
= receiver가 sender를 조절
= receiver buffer overflow 방지
```

### 16-6. 중요한 구분
- Flow Control: **수신자 처리 능력 문제**
- Congestion Control: **네트워크 자체 수용 능력 문제**

---

## 17. TCP Connection Management

### 17-1. 왜 연결 설정이 필요한가?
TCP는 데이터를 그냥 던지는 프로토콜이 아니라
- 상대가 살아있는지
- 서로 연결할 의사가 있는지
- 초기 sequence number가 무엇인지
- 상태를 어떻게 잡을지

를 먼저 맞춰야 한다.

---

## 18. 왜 2-Way Handshake가 위험한가

### 18-1. 문제
현실 네트워크는
- 지연이 가변적이고
- 중복 메시지가 생길 수 있고
- 재전송이 있고
- 순서가 바뀔 수 있다

### 18-2. 대표 문제: half-open connection
한쪽은 연결이 열렸다고 생각하지만  
다른 쪽은 아니거나 이미 종료된 상태일 수 있음

### 18-3. 결과
- 자원 낭비
- 잘못된 연결 유지
- duplicate data 문제

### 18-4. 핵심 문장
**2-way handshake는 서로의 상태를 충분히 확인하지 못한다.**

---

## 19. TCP 3-Way Handshake

### 19-1. 순서
1. Client → Server: SYN, Seq = x
2. Server → Client: SYN + ACK, Seq = y, ACK = x+1
3. Client → Server: ACK = y+1

### 19-2. 본질
이건 단순히 "메시지 3번 왕복"이 아니라,
**양방향 모두 초기 상태를 서로 확인하는 절차**다.

### 19-3. 손필기용 한 줄
```text
SYN -> SYN+ACK -> ACK
요청 -> 응답 -> 확인
```

### 19-4. 기억 포인트
- client가 server의 생존 확인
- server가 client의 생존 확인
- 서로 initial seq number 합의

---

## 20. TCP Closing

### 20-1. 기본 흐름
- 각 방향은 독립적으로 닫을 수 있음
- FIN 보내고 ACK 받음
- 양쪽 모두 닫히면 종료

### 20-2. 해석
연결은 양방향이라서
**닫는 것도 한쪽이 일방적으로 "끝"이라고 말하는 걸로 완성되지 않는다.**

---

## 21. Congestion의 본질

### 21-1. 혼잡이란?
너무 많은 sender가 너무 빠르게 보내서  
네트워크가 감당하지 못하는 상태

### 21-2. 나타나는 현상
- 라우터 큐 길어짐
- delay 증가
- 버퍼 overflow
- packet loss
- retransmission 증가
- 불필요한 duplicate까지 증가
- 결국 유효 throughput 감소

### 21-3. 핵심 구분
```text
Flow Control = receiver 보호
Congestion Control = network 보호
```

---

## 22. 혼잡 시나리오 1, 2, 3의 핵심 해석

## 22-1. 시나리오 1
- 버퍼 무한대 가정
- 손실은 없어도 입력률이 한계에 가까워질수록 지연이 급증

핵심:
**용량 근처로 갈수록 delay가 폭발한다.**

## 22-2. 시나리오 2
- 버퍼 유한
- 패킷 드롭 발생
- sender가 재전송
- transport layer input은 원본 + 재전송까지 포함

핵심:
**혼잡은 단순 느림이 아니라, 재전송 때문에 네트워크를 더 악화시키는 악순환이 된다.**

## 22-3. realistic scenario
- premature timeout 때문에 사실 필요 없는 duplicate 재전송까지 생김

핵심:
**혼잡이 심할수록 네트워크 자원이 낭비되고, 실제 유효 throughput이 줄어든다.**

## 22-4. 시나리오 3
- 다중 홉
- 어떤 패킷은 아래쪽에서 버려졌지만 위쪽 링크 자원은 이미 써버린 상태

핵심:
**뒤에서 버려진 패킷 때문에 앞단의 전송 자원도 통째로 낭비된다.**

---

## 23. 혼잡 제어 접근 방식

## 23-1. End-to-End congestion control
- 라우터가 직접 말해주지 않음
- sender가 loss, delay 같은 간접 신호를 보고 추정
- TCP의 기본 철학

## 23-2. Network-assisted congestion control
- 라우터가 직접 피드백
- 예: ECN, ATM, DECbit
- 현실에서는 모든 라우터 지원이 필요해 널리 단순하게 쓰이기 어려움

### 손필기 요약
```text
TCP 기본 = end-to-end 추정
ECN = 네트워크가 직접 알려주는 보조 방식
```

---

## 24. TCP Congestion Control의 핵심 변수: cwnd

### 24-1. cwnd란?
- congestion window
- sender가 동시에 네트워크에 띄워둘 수 있는 데이터 양

### 24-2. 전송 제한
```text
LastByteSent - LastByteAcked < cwnd
```

### 24-3. 의미
sender는 마음대로 무한정 보내는 게 아니라  
**cwnd가 허용하는 범위 안에서만 보낸다.**

---

## 25. AIMD

### 25-1. Additive Increase
- 혼잡이 없다고 보이면
- RTT마다 조금씩 선형 증가

### 25-2. Multiplicative Decrease
- 손실이 감지되면
- 전송률/cwnd를 크게 줄임(대표적으로 절반)

### 25-3. 왜 이렇게 하나?
- 네트워크 용량을 미리 정확히 모르므로
- 조금씩 탐색하다가
- 혼잡 신호가 보이면 과감히 줄이는 방식

### 25-4. 장점
- 안정성
- 효율성
- 공정성

### 25-5. 외우기
**조금씩 올리고, 문제 생기면 크게 내린다.**

---

## 26. Slow Start

### 26-1. 왜 이름은 slow인데 빠르게 늘어나나?
처음엔 작은 값에서 시작하기 때문에 "slow"지만  
증가 방식은 사실 **지수 증가**다.

### 26-2. 규칙
- 초기 cwnd = 1 MSS
- ACK 받을 때마다 cwnd 증가
- 결과적으로 RTT마다 대략 2배 증가

### 26-3. 목적
초기 네트워크 용량을 빠르게 탐색

### 26-4. 전환 시점
- 손실 발생하거나
- ssthresh에 도달하면
- congestion avoidance(선형 증가)로 넘어감

---

## 27. ssthresh

### 27-1. 의미
- slow start threshold
- 지수 증가를 멈추고 선형 증가로 바꾸는 기준점

### 27-2. 일반적 처리
손실 발생 시
```text
ssthresh = 손실 직전 cwnd / 2
```

---

## 28. TCP Tahoe / Reno / CUBIC 흐름 감각

### 28-1. Tahoe
- timeout 손실 시 강하게 줄임
- 보수적, 안정적
- 성능은 답답할 수 있음

### 28-2. Reno
- triple duplicate ACK 상황에서 fast retransmit / fast recovery
- Tahoe보다 좀 더 공격적이고 효율적

### 28-3. CUBIC
- 현대 Linux 기본
- 이전 손실 시점의 Wmax를 기준으로
- 초반엔 더 빠르게, 근처에선 더 조심스럽게 증가

### 28-4. 기억 포인트
```text
Tahoe: 확 줄이고 다시 시작
Reno: 중간 회복을 더 잘함
CUBIC: 현대형, 고속 환경에 유리
```

---

## 29. Bottleneck Link 관점

### 29-1. 왜 bottleneck이 중요한가?
전체 경로에서 가장 좁은 곳이 전체 throughput을 사실상 결정한다.

### 29-2. 핵심 인사이트
- bottleneck이 이미 꽉 찼다면
- sender가 더 빨리 보내도 end-to-end throughput은 거의 늘지 않는다
- 대신 RTT와 queueing delay만 커진다

### 29-3. 외우기 좋은 문장
**목표는 파이프를 딱 채우는 것, 넘치게 채우는 것이 아니다.**

---

## 30. Delay-based 접근, ECN, Fairness

## 30-1. Delay-based congestion control
- 손실을 일부러 유발하지 않고
- RTT 증가 같은 지연 신호로 혼잡을 감지
- 예: BBR 계열 아이디어와 연결해 생각 가능

## 30-2. ECN
- 라우터가 IP 헤더 비트로 혼잡 표시
- receiver가 ACK에 ECE 비트로 sender에게 알림

## 30-3. TCP fairness
이상적으로는 K개의 TCP 세션이 같은 병목 링크 R을 공유하면  
각자 평균 R/K를 가져가는 것이 목표다.

주의:
- RTT가 다르면 공정성 깨질 수 있음
- 앱이 여러 TCP 연결을 동시에 열면 더 많은 몫을 가져갈 수 있음
- UDP는 TCP처럼 rate throttling을 하지 않을 수 있음

---

## 31. QUIC / HTTP3

### 31-1. 왜 등장했는가?
전통적 HTTP/2 over TCP는
- TCP HOL blocking 문제
- TCP + TLS가 직렬 handshake
- 성능/유연성 한계

### 31-2. QUIC의 방향
- UDP 위에서 동작
- 신뢰성, 혼잡 제어, 보안, 다중 스트림을 애플리케이션 쪽으로 끌어올림

### 31-3. 장점
- handshake를 더 빠르게 처리
- stream별 병렬성 강화
- HOL blocking 완화
- HTTP/3의 기반

### 31-4. 한 줄 요약
**QUIC은 UDP 위에 TCP+TLS의 좋은 기능을 재구성한 현대형 전송 방식으로 보면 된다.**

---

## 32. 손필기용 대비표

| 구분 | 핵심 질문 | 답 |
|---|---|---|
| Network vs Transport | 어디까지 책임지나? | Network는 host, Transport는 process |
| UDP vs TCP | 연결/신뢰성 여부 | UDP는 없음, TCP는 있음 |
| Seq vs ACK | 무엇을 뜻하나? | Seq는 내가 보내는 시작 바이트, ACK는 네가 다음에 보낼 바이트 |
| Flow vs Congestion | 누구를 보호하나? | Flow는 receiver, Congestion은 network |
| GBN vs SR | 재전송 범위 | GBN은 뒤까지 전부, SR은 필요한 것만 |
| 2-way vs 3-way | 왜 3-way? | half-open, 중복 문제 방지 위해 |
| Timeout vs Fast Retransmit | 언제 재전송? | timeout 또는 triple duplicate ACK |
| Slow Start vs Congestion Avoidance | 증가 방식 | 전자는 지수 증가, 후자는 선형 증가 |
| TCP vs QUIC | 기반 | TCP는 전통 transport, QUIC은 UDP 위 현대형 구조 |

---

## 33. 시험 직전 체크리스트

아래를 설명 가능하면 Chapter 3의 핵심 뼈대는 잡힌 것이다.

- 전송 계층이 네트워크 계층과 다른 점을 설명할 수 있는가?
- multiplexing / demultiplexing을 포트 번호 중심으로 설명할 수 있는가?
- UDP를 왜 굳이 쓰는지 설명할 수 있는가?
- rdt가 왜 checksum, ACK, seq, timer를 필요로 하는지 설명할 수 있는가?
- Stop-and-Wait가 왜 비효율적인지 설명할 수 있는가?
- GBN과 SR의 차이를 설명할 수 있는가?
- TCP의 sequence number와 ACK number를 바이트 기준으로 설명할 수 있는가?
- EstimatedRTT, DevRTT, TimeoutInterval 관계를 말할 수 있는가?
- Flow control과 congestion control 차이를 명확히 말할 수 있는가?
- 왜 TCP가 3-way handshake를 쓰는지 말할 수 있는가?
- AIMD, slow start, ssthresh의 흐름을 말할 수 있는가?
- QUIC/HTTP3가 왜 나왔는지 말할 수 있는가?

---

## 34. 손으로 옮길 때 추천 배치

### 34-1. 1페이지
- 큰 그림
- transport vs network
- multiplex / demultiplex
- UDP vs TCP 비교표

### 34-2. 2페이지
- rdt1.0 ~ 3.0
- Stop-and-Wait
- pipelining
- GBN vs SR

### 34-3. 3페이지
- TCP segment 구조
- seq / ack
- RTT / timeout
- fast retransmit

### 34-4. 4페이지
- flow control
- 3-way handshake / closing
- congestion vs flow
- AIMD / slow start / ssthresh / CUBIC / QUIC

---

## 35. 그림 넣기 좋은 위치 표시

아래 항목은 손필기 때 간단 그림을 넣으면 기억이 잘 남는다.

- `||| transport vs network 비유 그림`
- `||| UDP demux vs TCP 4-tuple demux 그림`
- `||| Stop-and-Wait vs pipelining 타임라인`
- `||| GBN 윈도우 그림`
- `||| SR 윈도우 그림`
- `||| TCP seq / ACK 예시 그림`
- `||| 3-way handshake 그림`
- `||| flow control의 rwnd 그림`
- `||| congestion scenario 1~3 간단 큐 그림`
- `||| AIMD 톱니 그래프`
- `||| slow start -> congestion avoidance 전환 그래프`
- `||| QUIC 다중 stream 그림`

---

## 36. 마지막 암기 문장

- **전송 계층은 프로세스 간 통신을 담당한다.**
- **UDP는 단순함, TCP는 신뢰성과 제어가 강점이다.**
- **신뢰성은 checksum + ACK + seq + timer의 조합으로 만든다.**
- **GBN은 많이 다시 보내고, SR은 필요한 것만 다시 보낸다.**
- **TCP ACK는 다음에 원하는 바이트 번호다.**
- **Flow control은 receiver 보호, congestion control은 network 보호다.**
- **3-way handshake는 서로 살아 있고 준비됐음을 양방향으로 확인하는 절차다.**
- **혼잡 제어는 loss와 delay를 보며 보수적으로 추정하는 게임이다.**
- **AIMD는 조금씩 올리고 크게 줄인다.**
- **HTTP/3/QUIC은 UDP 위에 현대적으로 재구성된 전송 방식이다.**

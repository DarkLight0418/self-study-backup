# Chapter 3 Transport Layer 손필기 전사용 노트

## 목적

- 시험 직전 복습 가능
- 손으로 전사 가능
- 교안 + TCP 손필기 + 수업 강조 포인트 통합
- 비교형 / 서술형 / 흐름 설명 문제 대비

## 이 노트의 사용 방식

- 먼저 외울 핵심 문장부터 본다.
- 그 다음 개념별 구조를 본다.
- 헷갈리는 비교를 마지막에 정리한다.
- ASCII 도식은 손으로 그대로 옮겨 그린다.
- 공식은 **식 + 의미 + 왜 필요한지**까지 같이 외운다.

---

# 0. 먼저 외울 핵심 문장

- 전송 계층은 **호스트 대 호스트가 아니라 프로세스 대 프로세스 논리 통신**을 제공한다.
- 네트워크 계층이 집까지 보내는 역할이라면, 전송 계층은 **그 집 안에서 누구 방으로 갈지 분류**하는 역할이다.
- 멀티플렉싱은 여러 소켓의 데이터를 모아 세그먼트로 보내는 것이고, 디멀티플렉싱은 받은 세그먼트를 올바른 소켓으로 올려보내는 것이다.
- UDP는 **연결 설정이 없고, 단순하고, 빠르지만, 신뢰성과 순서 보장이 없다.**
- UDP도 checksum으로 오류 검출은 하지만, **완전한 신뢰성 보장은 하지 않는다.**
- Reliable Data Transfer의 핵심은 **오류 검출 + ACK + 재전송 + 순서 번호 + 타이머**의 조합이다.
- stop-and-wait는 이해는 쉽지만, RTT가 크면 **링크 활용도가 매우 낮다.**
- 파이프라이닝은 ACK를 기다리지 않고 여러 패킷을 띄워서 **성능을 끌어올리는 방식**이다.
- GBN은 하나가 빠지면 **그 뒤를 몰아서 다시 보내는 방식**이고, SR은 **빠진 것만 골라 다시 보내는 방식**이다.
- TCP는 **연결형, 신뢰적, 순서 보장, 바이트 스트림, full duplex** 프로토콜이다.
- TCP의 sequence number는 **세그먼트 번호가 아니라 바이트 번호**를 센다.
- TCP ACK 번호는 **다음에 받고 싶은 바이트 번호**다.
- TCP는 RTT가 변하므로 timeout도 고정값이 아니라 **EstimatedRTT와 DevRTT로 동적으로 잡는다.**
- TCP는 누적 ACK를 쓰므로, 중간 ACK 손실이 있어도 **나중 ACK 하나로 이전 것까지 커버**할 수 있다.
- fast retransmit은 **타임아웃까지 기다리지 않고, 중복 ACK 3개로 손실을 추정해 재전송**하는 것이다.
- Flow Control은 **수신자 버퍼 보호**이고, Congestion Control은 **네트워크 전체 보호**다.
- rwnd는 receive window이고, cwnd는 congestion window이다. 둘은 목적이 다르다.
- 2-way handshake는 half-open connection 같은 문제가 생길 수 있어서, TCP는 **3-way handshake**를 쓴다.
- 혼잡은 단순히 느린 게 아니라, **네트워크가 감당 가능한 양을 넘어서 큐 지연과 손실이 커지는 상태**다.
- AIMD는 **천천히 늘리고(Additive Increase), 손실 시 크게 줄이는(Multiplicative Decrease)** TCP의 핵심 혼잡 제어 아이디어다.
- slow start는 이름은 slow지만 실제로는 **초기에는 지수적으로 빠르게 증가**한다.
- bottleneck link가 꽉 차면 송신률을 더 올려도 처리량은 안 늘고, **지연만 증가**할 수 있다.
- QUIC/HTTP3는 UDP 위에서 동작하면서 **보안, 연결 설정, 신뢰성, 혼잡 제어를 더 유연하게 처리**한다.

---

# 1. 전송 계층의 큰 그림 (p2 ~ p9)

## 1-1. 정의 / 큰 그림

- 전송 계층의 목표
  - 멀티플렉싱 / 디멀티플렉싱 이해
  - 신뢰적 데이터 전달 이해
  - flow control 이해
  - congestion control 이해
  - 인터넷 전송 계층 프로토콜인 UDP, TCP 이해

- 핵심 정의
  - 네트워크 계층: **호스트와 호스트 사이**의 논리 통신
  - 전송 계층: **프로세스와 프로세스 사이**의 논리 통신

- 송신 측 동작
  - 응용 계층 메시지를 받음
  - 헤더 필드 값을 정함
  - 세그먼트를 만듦
  - IP 계층으로 넘김

- 수신 측 동작
  - IP 계층으로부터 세그먼트를 받음
  - 헤더를 확인함
  - 응용 계층 메시지를 꺼냄
  - 소켓 기준으로 올바른 프로세스로 전달함

- 인터넷 전송 계층의 두 축
  - TCP
    - reliable
    - in-order
    - flow control
    - congestion control
    - connection setup
  - UDP
    - unreliable
    - unordered
    - 단순함
    - 연결 설정 없음

- 둘 다 못 해주는 것
  - delay guarantee 보장 X
  - bandwidth guarantee 보장 X

## 1-2. 손필기식 표현

- 네트워크 계층은 **집까지 우편을 보내는 것**
- 전송 계층은 **집 안에서 누구 방으로 갈지 정하는 것**
- 즉, 집 주소만으로는 부족하고 **누구한테 줄지까지 알아야 함**

## 1-3. 시험 포인트

- "전송 계층과 네트워크 계층의 차이" 자주 나옴
- **host-to-host vs process-to-process** 비교 중요
- TCP/UDP는 서비스가 다르지, 단순히 빠르다/느리다로 외우면 틀리기 쉬움

## 1-4. ASCII 와이어프레임 또는 표

```text
응용 메시지
   ↓
[전송 계층]
- 헤더 추가
- 세그먼트 생성
   ↓
[네트워크 계층 IP]
   ↓
[링크/물리 계층]
   ↓
네트워크 전달
   ↓
[수신 측 네트워크 계층]
   ↓
[수신 측 전송 계층]
- 헤더 확인
- 소켓 분류
- 응용으로 전달
```

```text
비유
집 = 호스트
방 = 프로세스
편지 = 응용 메시지
우체국 = 네트워크 계층
집 안에서 누구 방인지 분류 = 전송 계층
```

---

# 2. 멀티플렉싱 / 디멀티플렉싱 (p10 ~ p22)

## 2-1. 정의 / 큰 그림

- 멀티플렉싱
  - 송신 측에서 여러 소켓의 데이터를 모음
  - 각 데이터에 전송 계층 헤더를 붙임
  - 세그먼트로 만들어 아래 계층으로 보냄

- 디멀티플렉싱
  - 수신 측에서 세그먼트를 받음
  - 헤더의 IP 주소 / 포트 번호를 봄
  - 어느 소켓에 넘길지 결정함

- 세그먼트에는 보통 이런 정보가 있음
  - source port
  - destination port
  - payload
  - 기타 헤더 필드

- 핵심 질문
  - "받은 세그먼트를 누구에게 줄 것인가?"
  - 정답: **IP 주소와 포트 번호 조합**

## 2-2. UDP에서의 디멀티플렉싱

- UDP는 connectionless
- 수신 측은 **destination port 번호**를 보고 소켓을 찾음
- 같은 목적지 포트로 오는 UDP 데이터그램은
  - source IP가 달라도
  - source port가 달라도
  - 같은 목적지 포트면 **같은 소켓으로 들어갈 수 있음**

- 즉 UDP는 단순함
  - "도착 포트 번호만 보면 됨"

## 2-3. TCP에서의 디멀티플렉싱

- TCP는 connection-oriented
- TCP 소켓은 4-tuple로 식별함
  - source IP
  - source port
  - destination IP
  - destination port

- 즉 같은 서버의 80번 포트로 들어와도
  - 클라이언트 IP/포트가 다르면
  - 서로 다른 TCP 연결로 구분 가능

- 서버가 여러 클라이언트를 동시에 받는 이유
  - 목적지는 같아도 출발지가 다르기 때문
  - 그래서 서버는 클라이언트마다 별도 연결 소켓을 가질 수 있음

## 2-4. 손필기식 표현

- UDP: **문 앞에 적힌 호수만 보고 넣음**
- TCP: **누가 보냈는지, 누구에게 가는지까지 다 보고 분류함**
- TCP는 단순 우편함이 아니라 **대화 상대별로 따로 창구를 여는 느낌**

## 2-5. 시험 포인트

- UDP 디멀티플렉싱: destination port 중심
- TCP 디멀티플렉싱: 4-tuple 중심
- "왜 웹 서버 1개가 여러 클라이언트와 동시에 통신 가능한가?"
  - 4-tuple로 연결을 구분하기 때문

## 2-6. ASCII 와이어프레임 또는 표

```text
[송신 측 멀티플렉싱]
App1 ─┐
App2 ─┼─> Transport Layer -> 헤더 추가 -> Segment 전송
App3 ─┘

[수신 측 디멀티플렉싱]
Segment 수신 -> 헤더 확인 -> 맞는 Socket 선택 -> App 전달
```

```text
UDP 분류 기준
[dest port]만 확인

TCP 분류 기준
[source IP, source port, dest IP, dest port]
4개를 모두 본다.
```

```text
비교
UDP : 같은 목적지 포트면 같은 소켓으로 갈 수 있음
TCP : 같은 목적지 포트라도 출발지 정보 다르면 다른 소켓으로 감
```

---

# 3. UDP (p23 ~ p35)

## 3-1. 정의 / 큰 그림

- UDP = User Datagram Protocol
- 특징
  - no frills
  - bare bones
  - best effort
  - connectionless

- 제공하는 것
  - 프로세스 간 데이터 전달 통로
  - 간단한 checksum 기반 오류 검출

- 제공하지 않는 것
  - 신뢰성 보장 X
  - 순서 보장 X
  - 연결 설정 X
  - flow control X
  - congestion control X

- 왜 쓰는가?
  - 연결 설정 지연이 없음
  - 헤더가 작음
  - 상태 유지가 단순함
  - 응용이 직접 속도를 조절하고 싶을 때 유리함
  - 일부 손실을 감수해도 되는 앱에 적합함

- 주 사용 예시
  - streaming multimedia
  - DNS
  - SNMP
  - HTTP/3의 기반 전송

## 3-2. UDP 세그먼트 구조

- 헤더 필드
  - source port
  - destination port
  - length
  - checksum

- length
  - UDP 세그먼트 전체 길이
  - 헤더 + 데이터 포함

- checksum
  - 전송 중 비트 뒤집힘 같은 오류 검출 용도

## 3-3. UDP checksum

- 목표
  - 전송 중 오류 검출

- 송신 측
  - UDP 세그먼트 내용과 일부 관련 정보를 16비트 단위로 봄
  - 1의 보수 덧셈 수행
  - 결과를 checksum 필드에 넣음

- 수신 측
  - 같은 방식으로 다시 계산
  - 계산값이 맞는지 비교
  - 다르면 오류 검출

- 주의
  - checksum이 같다고 해서 **오류가 절대 없다는 뜻은 아님**
  - 즉, 약한 보호 장치일 수 있음

## 3-4. 손필기식 표현

- UDP는 **일단 보내고 본다**에 가까움
- 빠르고 단순하지만, **잘 갔는지까지 책임지지는 않음**
- checksum은 **문서 훼손 검사 정도**이지, 택배 추적/재배송 시스템은 아님

## 3-5. 시험 포인트

- UDP가 connectionless인 이유 설명 가능해야 함
- "왜 UDP가 존재하는가?" 자주 나옴
- checksum은 신뢰성 전체가 아니라 **오류 검출 일부**라는 점 중요
- HTTP/3는 UDP 위에 필요한 기능을 얹는다는 연결까지 기억

## 3-6. ASCII 와이어프레임 또는 표

```text
UDP header
+----------------------+----------------------+
| source port          | destination port     |
+----------------------+----------------------+
| length               | checksum             |
+---------------------------------------------+
| data (payload)                             ...
+---------------------------------------------+
```

```text
UDP 성격
빠름 / 단순함 / 상태 적음
하지만
신뢰성 X / 순서 보장 X / 혼잡 제어 X / 흐름 제어 X
```

```text
언제 유리한가?
- 조금 잃어도 되는 실시간 앱
- RTT 아끼고 싶은 경우
- 앱이 직접 제어하고 싶은 경우
```

---

# 4. Reliable Data Transfer 원리 (p36 ~ p75)

## 4-1. 정의 / 큰 그림

- 목표
  - 아래 채널은 불안정해도
  - 위 응용 계층에는 **신뢰적인 채널처럼 보이게 만들기**

- 핵심 아이디어
  - 오류 검출
  - ACK / NAK
  - 재전송
  - sequence number
  - timer
  - window / buffering

- 중요한 관점
  - 송신자와 수신자는 서로 상태를 직접 볼 수 없음
  - 결국 메시지로 확인해야 함

## 4-2. rdt 1.0

- 가정
  - 채널이 완벽함
  - 비트 오류 없음
  - 패킷 손실 없음

- 동작
  - 송신자는 보내면 끝
  - 수신자는 받으면 전달

- 의미
  - 가장 이상적인 출발점
  - 현실과는 거리가 멀다

## 4-3. rdt 2.0

- 가정
  - 패킷이 손상될 수 있음
  - 손실은 아직 없음

- 추가 장치
  - checksum으로 오류 검출
  - ACK: 잘 받았음
  - NAK: 깨졌으니 다시 보내라

- 방식
  - stop-and-wait
  - 패킷 하나 보내고 응답을 기다림

- 문제
  - ACK/NAK 자체가 손상되면?
  - 송신자는 상대가 제대로 받았는지 알 수 없음

## 4-4. rdt 2.1

- 해결 아이디어
  - sequence number 추가
  - 중복 패킷을 구분 가능하게 함

- 왜 0, 1 두 개면 충분한가?
  - stop-and-wait라서 동시에 outstanding packet이 하나뿐이기 때문
  - 현재 패킷과 직전 패킷만 구분하면 됨

- 수신자 역할
  - 기대하던 seq인지 확인
  - 중복이면 버리고 적절히 응답

## 4-5. rdt 2.2

- NAK 없는 버전
- NAK 대신
  - 마지막으로 정상 수신한 패킷의 ACK를 다시 보냄
- 송신자 입장에서는
  - 중복 ACK를 받으면 현재 패킷 재전송으로 해석 가능

- 시험 포인트
  - TCP는 NAK-free 방식이라는 점과 연결해두면 좋음

## 4-6. rdt 3.0

- 가정 확장
  - 비트 오류도 있음
  - 패킷 손실도 있음

- 추가 장치
  - timer
  - timeout 시 재전송

- 핵심
  - ACK가 안 오면
    - 패킷이 사라졌거나
    - ACK가 사라졌거나
    - 그냥 늦게 오는 것일 수 있음
  - 그래서 sequence number가 꼭 필요함
  - 재전송이 중복이어도 수신자는 구분 가능해야 함

## 4-7. stop-and-wait의 한계

- 장점
  - 직관적
  - 설계가 단순

- 단점
  - 한 번 보내고 ACK를 기다리므로 링크가 놀 때가 많음
  - RTT가 큰 환경에서는 매우 비효율적

- 결론
  - 성능이 안 좋다
  - 그래서 pipelining이 필요함

## 4-8. pipelining

- idea
  - ACK를 하나씩 기다리지 말고
  - 여러 패킷을 동시에 in-flight 상태로 둔다

- 효과
  - utilization 증가
  - 링크를 더 잘 활용 가능

- 대신 필요한 것
  - 더 큰 sequence number 공간
  - sender / receiver buffer
  - window 관리

## 4-9. Go-Back-N (GBN)

- 의미
  - sender가 window 크기 N만큼 여러 패킷을 연속 전송할 수 있는 pipelining 방식
  - 하나가 손실되면 **그 패킷부터 뒤를 다시 보내는 방식**

- sender
  - 윈도우 크기 N
  - ACK는 cumulative ACK
  - `send_base` = 가장 오래된 미확인 패킷
  - `nextseqnum` = 다음에 보낼 패킷 번호
  - 가장 오래된 미확인 패킷 기준으로 타이머 하나 사용

- timeout 발생 시
  - `send_base`부터 `nextseqnum - 1`까지 **몰아서 재전송**

- receiver
  - in-order만 정상 처리
  - out-of-order는 보통 버림
  - 가장 마지막 in-order 패킷 기준으로 ACK 재전송

- 왜 비효율적인가?
  - 패킷 하나만 빠져도 뒤 패킷까지 다시 보내야 할 수 있음

- 장점
  - 구현 단순
- 단점
  - 하나 빠지면 뒤도 다 다시 보내게 되어 비효율 가능

## 4-10. Selective Repeat (SR)

- sender
  - 패킷별 ACK 관리
  - 패킷별 타이머 관리
  - 손실된 것만 골라 재전송

- receiver
  - out-of-order도 buffer에 저장 가능
  - 각 패킷 개별 ACK
  - 빠진 것이 채워지면 그때 순서대로 상위 계층에 전달

- 장점
  - 재전송 낭비 감소
- 단점
  - 구현 복잡
  - 버퍼링 / 윈도우 관리 더 어렵다

## 4-11. SR의 딜레마

- sequence number 공간이 너무 작고
- window가 너무 크면
- 오래된 패킷과 새로운 패킷을 구분 못 할 수 있음

- 그래서 필요 조건
  - **window size <= sequence number space / 2**

- 왜 중요한가?
  - 수신자는 송신자 내부 상태를 직접 못 보기 때문
  - 번호 재사용 시 혼동이 생기면 치명적

## 4-12. 손필기식 표현

- rdt의 본질은
  - **잘 갔니? -> 응 / 아니 / 모르겠음 -> 그럼 다시**
- stop-and-wait는
  - **한 장 보내고 답장 기다리기**
- pipelining은
  - **여러 장을 연속 투입해서 길을 비워두지 않는 것**
- GBN은
  - **하나 꼬이면 뒤도 줄줄이 다시**
- SR은
  - **빠진 것만 콕 집어 다시**

## 4-13. 시험 포인트

- rdt2.0의 치명적 문제: ACK/NAK 손상
- rdt2.1이 seq 번호를 도입하는 이유
- rdt2.2는 NAK 없이 ACK만으로 동작
- rdt3.0은 loss 대응 위해 timeout 도입
- stop-and-wait와 pipelining 성능 비교
- GBN vs SR 비교는 매우 자주 나옴
- SR의 window 제약도 단답형으로 나올 수 있음

## 4-14. ASCII 와이어프레임 또는 표

```text
Reliable Data Transfer 발전 흐름
rdt1.0 : 채널 완벽
rdt2.0 : 비트 오류 대응 (ACK/NAK)
rdt2.1 : ACK/NAK 손상 대응 (seq 추가)
rdt2.2 : NAK 없이 ACK만 사용
rdt3.0 : 손실 대응 (timer + timeout)
```

```text
stop-and-wait
Sender: pkt1 보냄 -> ACK 기다림 -> pkt2 보냄 -> ACK 기다림

문제
[보내는 시간] << [왕복 대기 시간]
=> 링크가 자주 논다.
```

```text
GBN
send: 0 1 2 3
loss:    2 손실
recv: 0 OK, 1 OK, 3 버림
ACK: ack1 반복
timeout -> 2,3,4,5 ... 다시 전송
```

```text
SR
send: 0 1 2 3
loss:    2 손실
recv: 0 OK, 1 OK, 3 buffer
ACK: ack0, ack1, ack3
timeout(2) -> 2만 재전송
recv: 2 도착 -> 2,3 순서 맞춰 전달
```

```text
GBN vs SR 핵심
GBN : 누적 ACK, 뒤도 몰아서 재전송
SR  : 개별 ACK, 필요한 것만 재전송
```

---

# 5. TCP 기본 구조와 특징 (p76 ~ p80)

## 5-1. 정의 / 큰 그림

- TCP는 connection-oriented transport
- 특징
  - reliable
  - in-order
  - byte stream
  - full duplex
  - point-to-point
  - cumulative ACK
  - pipelining 사용
  - flow control + congestion control 포함

- 바이트 스트림이라는 말의 의미
  - 메시지 경계가 없음
  - 응용이 보낸 바이트 흐름을 TCP가 나눠서 세그먼트화함
  - 즉 "메시지 단위"보다 "연속 바이트 흐름" 관점이 중요

## 5-2. TCP 세그먼트 구조

- 주요 필드
  - source port
  - destination port
  - sequence number
  - acknowledgement number
  - receive window(rwnd)
  - checksum
  - flags(SYN, FIN, RST, ACK 등)
  - options

- sequence number
  - 세그먼트 번호가 아니라
  - **이 세그먼트 데이터의 첫 바이트 번호**

- acknowledgement number
  - **다음에 받고 싶은 바이트 번호**
  - cumulative ACK

- rwnd
  - 수신 측이 지금 더 받아줄 수 있는 양

## 5-3. Telnet 예시 관점

- Host A가 문자 'C' 1바이트 보냄
- seq = 현재 바이트 번호
- Host B는 그 1바이트를 받고
  - ACK = 다음 기대 바이트 번호
  - 필요하면 데이터도 같이 보냄

- 즉 ACK는
  - "여기까지 받았음"이라기보다
  - "다음엔 이 번호부터 보내"로 이해하는 게 정확함

## 5-4. 손필기식 표현

- TCP는 **메시지 상자 번호**가 아니라 **바이트 번호표**를 붙인다.
- ACK는 **다음 거 보내라**는 번호다.
- 즉, "42번을 받았어"보다 "43번부터 줘"가 핵심이다.

## 5-5. 시험 포인트

- sequence number는 바이트 기준
- ACK number는 다음 기대 바이트 기준
- cumulative ACK 개념 필수
- TCP는 message boundary를 보장하지 않는다는 점도 종종 출제됨

## 5-6. ASCII 와이어프레임 또는 표

```text
TCP header (단순화)
+----------------------+----------------------+
| source port          | destination port     |
+---------------------------------------------+
| sequence number                            |
+---------------------------------------------+
| acknowledgement number                     |
+----------------------+----------------------+
| header len / flags   | receive window       |
+----------------------+----------------------+
| checksum             | urgent pointer       |
+---------------------------------------------+
| options (optional)                         |
+---------------------------------------------+
| data                                       |
+---------------------------------------------+
```

```text
seq / ack 감각
보낸 데이터 시작 번호 = seq
상대가 다음에 원하는 번호 = ack
```

---

# 6. TCP RTT, Timeout, ACK, 재전송 (p81 ~ p88)

## 6-1. RTT와 Timeout

- RTT = Round Trip Time
  - 작은 패킷이 갔다가 ACK가 돌아오는 왕복 시간
  - 고정값이 아님
  - 네트워크 상태 따라 변함

- SampleRTT
  - 실제 측정한 RTT 샘플
  - 단일 측정값만 믿으면 너무 출렁임

- EstimatedRTT
  - 최근 추세를 반영한 부드러운 추정값

- 공식
  - EstimatedRTT = (1 - α) × EstimatedRTT + α × SampleRTT
  - 보통 α = 0.125

- 의미
  - 과거값을 더 많이 반영
  - 현재 샘플도 일부 반영
  - 즉 지수 가중 이동 평균(EWMA)

- DevRTT
  - RTT 변동폭 추정치
  - DevRTT = (1 - β) × DevRTT + β × |SampleRTT - EstimatedRTT|
  - 보통 β = 0.25

- TimeoutInterval
  - TimeoutInterval = EstimatedRTT + 4 × DevRTT

- 왜 이렇게 하나?
  - timeout이 너무 짧으면 성급한 재전송
  - 너무 길면 손실 대응이 늦음
  - 그래서 평균 + 안전 마진으로 잡는다

## 6-2. TCP Sender의 기본 동작

- 응용에서 데이터가 오면
  - seq 번호를 가진 세그먼트 생성
  - 아직 타이머가 없으면 시작

- 타이머는 보통
  - **가장 오래된 unACKed 세그먼트 기준 1개**

- timeout 발생 시
  - 해당 세그먼트 재전송
  - 타이머 재시작

- ACK 수신 시
  - 누적 ACK 처리
  - 확인된 바이트 범위 갱신
  - 아직 미확인 데이터가 남아 있으면 타이머 유지 또는 재시작

## 6-3. TCP Receiver ACK 생성 규칙

- 정상 in-order 세그먼트 도착
  - 최대 500ms 정도 delayed ACK 가능
  - 다음 세그먼트도 같이 ACK할 수 있음

- in-order 세그먼트가 연속해서 오면
  - 즉시 누적 ACK 전송 가능

- out-of-order 세그먼트 도착
  - gap 감지
  - duplicate ACK 전송
  - "나는 아직 이 번호를 기대하고 있음"을 알림

- gap을 메우는 세그먼트 도착
  - 즉시 ACK

## 6-4. 재전송 시나리오

### 1) ACK 손실

- 데이터는 잘 갔음
- ACK만 사라짐
- sender는 timeout 후 재전송 가능
- receiver는 중복 데이터 감지 후 다시 ACK 가능

### 2) premature timeout

- 원래 ACK가 늦게 온 것뿐인데
- sender가 먼저 timeout으로 재전송할 수도 있음
- cumulative ACK가 이를 커버 가능

### 3) cumulative ACK의 장점

- 중간 ACK가 하나 손실되어도
- 더 큰 ACK가 오면 이전 바이트까지 한 번에 확인 가능

## 6-5. TCP fast retransmit

- 타임아웃까지 기다리면 느림
- 같은 ACK가 3번 더 오면
  - 뒤의 세그먼트들은 왔는데
  - 특정 세그먼트만 비었다는 뜻일 가능성이 큼
- 그래서 timeout 전에 미리 재전송

- 핵심 문장
  - **triple duplicate ACK -> 손실 추정 -> 바로 재전송**

## 6-6. 손필기식 표현

- SampleRTT는 막 튀어도 EstimatedRTT는 **흐름을 본다**
- Timeout은 그냥 RTT 하나가 아니라 **RTT + 흔들림 보정값**
- duplicate ACK가 쌓이면
  - **아, 중간 하나 비었구나** 하고 빨리 다시 보낸다

## 6-7. 시험 포인트

- EstimatedRTT / DevRTT / TimeoutInterval 공식
- delayed ACK, duplicate ACK 구분
- cumulative ACK의 장점
- fast retransmit이 timeout보다 빠른 이유

## 6-8. ASCII 와이어프레임 또는 표

```text
RTT 추정
SampleRTT  : 지금 측정한 값 (출렁임 큼)
EstimatedRTT : 부드러운 추세값
DevRTT : 흔들림 정도
Timeout = 평균 + 안전마진
```

```text
공식
EstimatedRTT = (1-a)EstimatedRTT + aSampleRTT
DevRTT       = (1-b)DevRTT + b|SampleRTT - EstimatedRTT|
Timeout      = EstimatedRTT + 4DevRTT
```

```text
fast retransmit 감각
Sender -> pkt100 보냄 (손실)
Sender -> pkt120 보냄
Sender -> pkt140 보냄
Receiver는 계속 "ACK=100" 보냄
같은 ACK 3개 누적
=> Sender: 100번 세그먼트 재전송
```

---

# 7. TCP Flow Control (p89 ~ p95)

## 7-1. 정의 / 큰 그림

- 문제 상황
  - 네트워크는 데이터를 잘 전달했는데
  - 수신 애플리케이션이 천천히 읽으면
  - TCP receive buffer가 넘칠 수 있음

- 해결
  - receiver가 sender를 제어
  - "내가 지금 얼마나 더 받을 수 있는지" 광고함

- 이것이 Flow Control
  - 목적: **receiver buffer overflow 방지**

## 7-2. 핵심 변수: rwnd

- rwnd = receive window
- TCP 헤더에 담겨 수신 측이 송신 측에 알려줌
- 의미
  - "내 버퍼에 지금 남은 공간"
  - "이만큼까지는 더 보내도 됨"

- sender는
  - in-flight 데이터 양이 rwnd를 넘지 않도록 조절함

- 결과
  - 수신 버퍼가 넘치지 않음

## 7-3. 버퍼 관점

- RcvBuffer = 수신 버퍼 전체 크기
- 이미 받은 데이터 중 아직 앱이 안 읽은 데이터가 차지하는 양이 있음
- 남은 공간이 바로 rwnd

- 요약
  - rwnd는 **수신자 처리 여력**의 표현

## 7-4. 왜 중요한가?

- 신뢰성만 보장해도 끝이 아님
- 상대가 감당 못 하면 결국 데이터 손실 / 성능 저하
- 그래서 TCP는 sender가 마음대로 막 보내지 않게 함

## 7-5. 손필기식 표현

- Flow Control은
  - **수신자가 송신자에게 브레이크를 거는 것**
- 핵심은 네트워크가 아니라
  - **상대 버퍼가 버틸 수 있느냐**
- rwnd는
  - **내가 지금 받아줄 수 있는 빈칸 수**라고 보면 됨

## 7-6. 시험 포인트

- Flow Control과 Congestion Control 절대 혼동 금지
- rwnd 의미 설명 가능해야 함
- sender가 rwnd 기반으로 in-flight 데이터 제한한다는 문장 중요

## 7-7. ASCII 와이어프레임 또는 표

```text
Receiver 쪽
+-----------------------------+
| 이미 받은 데이터           |
| (앱이 아직 안 읽음)        |
+-------------+---------------+
              | free space     |
              | = rwnd         |
              +---------------+
```

```text
Flow Control 핵심
Receiver: "지금 rwnd = 3000 byte야"
Sender  : "그럼 ACK 안 된 데이터 총량을 3000 byte 이내로 유지"
```

```text
구분
Flow Control = 수신자 보호
Congestion Control = 네트워크 보호
```

---

# 8. TCP Connection Management (p96 ~ p103)

## 8-1. 연결 설정이 왜 필요한가?

- TCP는 connection-oriented
- 데이터를 주고받기 전에
  - 연결을 맺을 의사가 있는지
  - 초기 sequence number는 무엇인지
  - 상태를 어떻게 둘지
  - 서로 살아 있는지
    를 확인해야 함

## 8-2. 2-way handshake가 위험한 이유

- 현실 네트워크는 이상적이지 않음
  - 지연 있음
  - 재전송 있음
  - 순서 뒤바뀜 가능

- 문제 1: half-open connection
  - 한쪽은 연결 끝났다고 생각
  - 다른 쪽은 아직 살아 있다고 생각
  - 자원 낭비 발생

- 문제 2: duplicate data acceptance
  - 오래된 요청/데이터가 뒤늦게 도착하면
  - 새 연결 데이터로 오인될 수 있음

## 8-3. 3-way handshake

- 1단계: client -> server
  - SYN, Seq = x
  - "연결하고 싶다. 내 시작 번호는 x"

- 2단계: server -> client
  - SYN + ACK, Seq = y, ACK = x+1
  - "나도 연결 가능. 내 시작 번호는 y. 네 SYN도 확인함"

- 3단계: client -> server
  - ACK, ACK = y+1
  - "네 SYN도 확인함"

- 의미
  - 요청 - 응답 - 확인
  - 양방향 초기 상태를 모두 확인

## 8-4. 연결 종료

- TCP는 양방향 full duplex라서
  - 각 방향이 독립적으로 닫힘

- 종료 기본 흐름
  - FIN 보냄
  - ACK 받음
  - 반대쪽도 FIN 보냄
  - ACK 응답

- 요점
  - 보내는 쪽을 먼저 닫아도
  - 반대 방향 데이터는 잠시 더 갈 수 있음

## 8-5. 손필기식 표현

- 3-way handshake는 단순히 3번 말하는 게 아니라
  - **양쪽이 서로 초기 상태를 확인하는 과정**
- 2-way로 끝내면
  - **반쪽만 열린 연결** 같은 위험이 남음
- 종료도 한 번에 뚝 끊는 게 아니라
  - **양 방향을 따로 닫는다**

## 8-6. 시험 포인트

- 2-way handshake가 왜 불충분한지 설명 가능해야 함
- half-open connection 사례 자주 나옴
- 3-way handshake 각 단계의 SYN / ACK 의미 구분
- 종료 시 FIN / ACK 흐름과 양방향 독립성 기억

## 8-7. ASCII 와이어프레임 또는 표

```text
TCP 3-way handshake
Client                              Server
  | ---- SYN, Seq=x ------------->  |
  | <--- SYN+ACK, Seq=y, Ack=x+1 -- |
  | ---- ACK, Ack=y+1 ----------->  |

=> 연결 성립
```

```text
왜 2-way가 위험한가?
오래된 요청이 늦게 도착
-> 서버는 새 연결로 오해 가능
-> client는 이미 끝났을 수도 있음
=> half-open / dup data 문제
```

```text
TCP closing
A -> FIN
B -> ACK
B -> FIN
A -> ACK
```

---

# 9. 혼잡 제어의 원리 (p104 ~ p117)

## 9-1. 혼잡이란?

- informal 정의
  - **너무 많은 송신자들이 너무 많은 데이터를 너무 빠르게 보내서 네트워크가 감당 못 하는 상태**

- 나타나는 현상
  - 라우터 버퍼 큐 지연 증가
  - 패킷 손실 증가
  - 재전송 증가
  - 실제 처리량 효율 저하

- 중요한 구분
  - Flow Control
    - 한 sender가 한 receiver에게 너무 빠름
  - Congestion Control
    - 네트워크 전체가 감당 못 함

## 9-2. 혼잡 시나리오 1

- 하나의 라우터, 두 흐름, 무한 버퍼라고 생각
- 입력률이 링크 용량에 가까워질수록
  - throughput은 한계에 가까워짐
  - delay는 급격히 증가

- 핵심 인사이트
  - 용량 근처에서는 **지연이 폭발적으로 늘 수 있음**

## 9-3. 혼잡 시나리오 2

- 라우터 버퍼가 유한함
- 패킷 드롭 발생 가능
- 손실된 패킷은 재전송됨

- 결과
  - 원래 데이터 + 재전송 데이터가 함께 링크를 점유
  - 같은 처리량을 얻기 위해 더 많은 일을 하게 됨

- 현실에서는 더 나쁨
  - timeout이 너무 빨라서 불필요한 duplicate 재전송도 생김
  - 유효 throughput이 더 떨어짐

## 9-4. 혼잡 시나리오 3

- 여러 홉, 여러 송신자
- 아래쪽에서 버려질 패킷을 위해
  - 위쪽 링크와 버퍼가 이미 낭비될 수 있음

- 의미
  - downstream drop이 생기면
  - upstream 자원도 허무하게 낭비됨

## 9-5. 혼잡 제어 접근 방식

### 1) End-to-End Congestion Control

- 네트워크가 직접 말해주지 않음
- sender가 loss / delay를 보고 추정함
- TCP의 기본 방식

### 2) Network-Assisted Congestion Control

- 라우터가 직접 피드백을 줌
- 혼잡 상태 표시 또는 전송률 힌트 제공
- 예: ECN, ATM, DECbit

## 9-6. 손필기식 표현

- 혼잡은 단순히 느린 게 아니라
  - **큐가 쌓이고, 떨어지고, 다시 보내고, 더 막히는 악순환**
- flow control은
  - **상대 버퍼 문제**
- congestion control은
  - **망 전체 문제**

## 9-7. 시험 포인트

- 혼잡의 정의를 지연 / 손실 / 재전송 관점에서 설명 가능해야 함
- scenario 2의 핵심: retransmission이 오히려 자원 낭비를 키움
- end-to-end vs network-assisted 비교 자주 나옴

## 9-8. ASCII 와이어프레임 또는 표

```text
보내는 속도 증가
   ↓
라우터 큐 증가
   ↓
지연 증가
   ↓
버퍼 overflow
   ↓
패킷 손실
   ↓
재전송 증가
   ↓
더 혼잡
```

```text
Flow vs Congestion
Flow Control       : receiver buffer 보호
Congestion Control : network 전체 보호
```

```text
혼잡 비용
- delay 증가
- loss 증가
- retransmission 증가
- duplicate 증가
- upstream 자원 낭비
- effective throughput 감소
```

---

# 10. TCP Congestion Control (p118 ~ p134)

## 10-1. AIMD (p119 ~ p120)

- AIMD = Additive Increase, Multiplicative Decrease

- Additive Increase
  - 손실이 없으면 RTT마다 조금씩 증가
  - 보통 1 MSS씩 증가하는 감각으로 이해

- Multiplicative Decrease
  - 손실이 감지되면 전송률 / 윈도우를 크게 줄임
  - 대표적으로 절반 수준으로 감소

- 왜 이 방식인가?
  - 네트워크 용량을 정확히 모르므로 보수적으로 탐색
  - 손실을 혼잡 신호로 해석
  - 안정성 / 효율성 / 공정성 측면에서 좋음

## 10-2. cwnd (p121)

- cwnd = congestion window
- 의미
  - sender가 네트워크 혼잡을 고려해서
  - 동시에 날려둘 수 있는 데이터 양

- sender 제약
  - 대략 in-flight 데이터가 cwnd 이내여야 함

- 전송률 감각
  - TCP rate ≈ cwnd / RTT

- 중요
  - rwnd는 수신자 관점
  - cwnd는 네트워크 관점

## 10-3. Slow Start (p122)

- 연결 시작 시
  - cwnd = 1 MSS에서 시작
  - ACK가 올 때마다 증가
  - 결과적으로 RTT마다 거의 2배씩 커짐

- 이름은 slow start지만
  - 실제 증가는 초반에 매우 빠름

- 목적
  - 처음부터 너무 크게 보내면 위험
  - 그렇다고 너무 천천히 탐색하면 비효율
  - 그래서 작게 시작하되 빠르게 키움

## 10-4. Congestion Avoidance와 ssthresh (p123 ~ p124)

- ssthresh = slow start threshold
- 손실 발생 시
  - ssthresh를 손실 직전 cwnd의 절반 정도로 설정

- 동작
  - cwnd < ssthresh
    - slow start 영역
    - 빠르게 증가
  - cwnd > ssthresh
    - congestion avoidance 영역
    - 선형 증가

- 의미
  - 초반에는 빠르게 빈 대역폭 탐색
  - 임계점 근처부터는 조심스럽게 증가

## 10-5. 손실 감지 시 반응

### 1) triple duplicate ACK

- Reno 계열 감각
  - 손실 추정
  - cwnd를 크게 줄이되 timeout만큼 심하게는 아님
  - 빠른 재전송 / 빠른 회복 사용

### 2) timeout

- 더 심각한 혼잡 신호로 해석
- cwnd를 1 MSS 수준까지 낮추는 감각
- 다시 slow start로 돌아감

## 10-6. Tahoe vs Reno

- Tahoe
  - 손실 시 강하게 줄임
  - 안정성은 좋지만 성능 손해 가능

- Reno
  - triple duplicate ACK 상황에서 더 완만한 회복
  - 실전 성능 개선

- 시험에서는 보통
  - timeout은 더 심각한 신호
  - triple duplicate ACK는 그보다는 덜 심각한 신호
    로 정리하면 좋다

## 10-7. TCP CUBIC (p125 ~ p126)

- 아이디어
  - 이전 손실 시점의 최대 윈도우 Wmax를 기준으로
  - 다시 그 근처까지 갈 때
    - 멀리 있을 때는 빠르게 증가
    - 가까워질수록 조심스럽게 증가

- 장점
  - classic TCP보다 고속 환경에서 throughput이 더 좋을 수 있음

- 특징
  - Linux 기본 TCP로 널리 사용

## 10-8. Bottleneck link 관점 (p127 ~ p128)

- 혼잡은 보통 병목 링크에서 터짐
- 병목 링크는 거의 항상 바쁨
- sender가 전송률을 더 올려도
  - 병목이 이미 꽉 찼다면 throughput은 안 늘 수 있음
  - 대신 RTT와 큐 지연만 증가

- 목표
  - **pipe를 딱 채우되, 더 넘치게 채우지 않는 것**

## 10-9. Delay-based 접근 / BBR (p129 ~ p130)

- loss를 기다리기 전에
  - RTT 증가 등을 보고 혼잡을 판단

- 핵심
  - RTTmin = 비혼잡 상태 최소 RTT
  - 현재 throughput이 비혼잡 기대치와 얼마나 차이 나는지 보고 판단

- 장점
  - 손실을 일부러 유도하지 않고도 혼잡 제어 가능
  - 지연을 낮게 유지하려는 방향

- 예시
  - BBR

## 10-10. ECN (p131)

- Explicit Congestion Notification
- 네트워크 보조 혼잡 제어

- 동작 감각
  - 라우터가 IP 헤더 비트로 혼잡 표시
  - 수신자가 ACK에서 ECE 비트 등으로 송신자에게 알림
  - 송신자는 이를 보고 전송률 조정

- 의미
  - 손실이 나기 전에 혼잡 신호를 전할 수 있음

## 10-11. TCP Fairness (p132 ~ p134)

- 이상적 목표
  - 같은 병목 링크를 K개 TCP 세션이 공유하면
  - 각자 대략 R/K 정도 평균 대역폭 사용

- 이상적인 조건에선 공정하다고 볼 수 있음
  - RTT 비슷
  - 세션 수 고정
  - 모두 congestion avoidance 중

- 그러나 현실 한계
  - UDP 앱은 TCP처럼 양보 안 할 수 있음
  - 한 앱이 병렬 TCP 연결 여러 개 열면 더 많이 가져갈 수 있음

## 10-12. 손필기식 표현

- cwnd는 **네트워크 눈치 보면서 sender가 스스로 정하는 보낼 양**
- slow start는 **처음엔 작게 시작하지만 실제론 꽤 빠르게 커진다**
- AIMD는 **조금씩 올려보다가 문제 생기면 확 줄이는 톱니 패턴**
- bottleneck이 꽉 찼으면 더 밀어 넣어도 소용없고 **지연만 늘어난다**

## 10-13. 시험 포인트

- AIMD 정의 정확히
- cwnd 의미와 rwnd와의 차이
- slow start vs congestion avoidance
- timeout vs triple duplicate ACK 반응 차이
- ssthresh 의미
- Tahoe / Reno / CUBIC 큰 흐름 비교
- fairness와 parallel TCP connection의 함정

## 10-14. ASCII 와이어프레임 또는 표

```text
AIMD 톱니 모양
rate
 ^        /\       /\       /\
 |       /  \     /  \     /  \
 |      /    \   /    \   /    \
 |_____/      \_/      \_/      \___> time
        조금씩 증가    손실 시 확 줄임
```

```text
slow start vs congestion avoidance
slow start           : 빠르게(지수적) 증가
congestion avoidance : 천천히(선형) 증가
기준                 : ssthresh
```

```text
cwnd / rwnd 구분
cwnd : 혼잡 때문에 sender가 정한 한도
rwnd : 수신 버퍼 때문에 receiver가 정한 한도
실제 전송 가능량은 둘 다 고려해야 함
```

```text
병목 링크 감각
Sender rate ↑
   ↓
Queue length ↑
   ↓
RTT ↑
   ↓
Loss ↑
   ↓
이제 더 올려도 throughput은 잘 안 늘 수 있음
```

```text
Tahoe / Reno / CUBIC
Tahoe : 손실 시 강하게 감소, 보수적
Reno  : dup ACK 기반 fast recovery 활용
CUBIC : Wmax 기준으로 더 유연하게 증가
```

---

# 11. 전송 계층의 진화: QUIC / HTTP3 (p135 ~ p140)

## 11-1. 큰 그림

- TCP와 UDP는 오랫동안 핵심 전송 프로토콜이었음
- 그런데 환경이 다양해짐
  - long fat pipe
  - wireless
  - long-delay links
  - data center
  - background traffic

- 그래서 일부 기능을 응용 계층 쪽으로 올려서
  - UDP 위에 더 유연하게 구현하는 방향이 등장

## 11-2. QUIC

- UDP 위의 application-layer protocol
- 목표
  - HTTP 성능 개선
  - 보안 / 연결 설정 / 신뢰성 / 혼잡 제어를 더 유연하게 처리

- 구조 감각
  - HTTP/2 over TCP + TLS
  - HTTP/3 over QUIC over UDP

## 11-3. 장점

- 여러 application stream을 하나의 QUIC connection 위에서 multiplex
- stream별 오류 제어 가능
- 공통 혼잡 제어 가능
- 보안 내장적 성격이 강함
- 연결 설정이 더 빠름
  - TCP handshake + TLS handshake를 따로 두는 대신
  - QUIC은 한 번의 handshake 쪽으로 단순화

## 11-4. HOL blocking 완화

- TCP 위에서는 한 stream 문제로 전체 전달이 막히는 HOL blocking 문제가 있을 수 있음
- QUIC은 stream 단위 병렬성이 더 좋아
  - 하나 문제 나도 다른 stream 영향이 줄어듦

## 11-5. 손필기식 표현

- QUIC은 **UDP 위에 필요한 걸 얹어서 TCP+TLS의 불편함을 줄이려는 방향**
- HTTP/3는 단순히 UDP 쓴다가 아니라
  - **UDP 위에서 신뢰성, 보안, 혼잡 제어까지 다시 설계한 것**

## 11-6. 시험 포인트

- QUIC이 UDP 위에 올라간다는 점
- HTTP/3와 QUIC의 관계
- handshake 단축, HOL blocking 완화 포인트

## 11-7. ASCII 와이어프레임 또는 표

```text
기존
HTTP/2
  ↓
TLS
  ↓
TCP
  ↓
IP

HTTP/3
  ↓
QUIC
  ↓
UDP
  ↓
IP
```

```text
QUIC 장점
- 연결 설정 빠름
- 보안 포함
- stream 병렬 처리
- HOL blocking 완화
- UDP 기반 유연성
```

---

# 마지막. 헷갈리는 비교 정리

## 1. TCP vs UDP

| 항목      | TCP             | UDP                             |
| --------- | --------------- | ------------------------------- |
| 연결 설정 | O               | X                               |
| 신뢰성    | O               | X                               |
| 순서 보장 | O               | X                               |
| 흐름 제어 | O               | X                               |
| 혼잡 제어 | O               | X                               |
| 헤더/구조 | 더 복잡         | 더 단순                         |
| 대표 감각 | 정확하고 관리형 | 빠르고 단순형                   |
| 예시      | 웹, 파일, 메일  | DNS, 일부 스트리밍, HTTP/3 기반 |

## 2. Flow Control vs Congestion Control

| 항목      | Flow Control            | Congestion Control    |
| --------- | ----------------------- | --------------------- |
| 보호 대상 | 수신자 버퍼             | 네트워크 전체         |
| 기준 변수 | rwnd                    | cwnd                  |
| 질문      | 상대가 더 받을 수 있나? | 망이 더 버틸 수 있나? |
| 문제 원인 | receiver가 느림         | 라우터/링크 혼잡      |

## 3. GBN vs SR

| 항목              | GBN                  | SR              |
| ----------------- | -------------------- | --------------- |
| ACK 방식          | 누적 ACK             | 개별 ACK        |
| out-of-order 처리 | 보통 버림            | 버퍼링 가능     |
| 재전송 범위       | 손실 이후 몰아서     | 손실된 것만     |
| 타이머            | 보통 oldest 기준 1개 | 패킷별          |
| 구현 난이도       | 낮음                 | 높음            |
| 효율              | 낮을 수 있음         | 더 좋을 수 있음 |

## 4. stop-and-wait vs pipelining

| 항목                  | stop-and-wait   | pipelining                     |
| --------------------- | --------------- | ------------------------------ |
| outstanding packet 수 | 1개             | 여러 개                        |
| 구현                  | 단순            | 복잡                           |
| 성능                  | RTT 크면 비효율 | 활용도 높음                    |
| 필요 요소             | 기본 ACK/재전송 | window, buffering, 큰 seq 공간 |

## 5. cumulative ACK vs duplicate ACK

| 항목 | cumulative ACK             | duplicate ACK              |
| ---- | -------------------------- | -------------------------- |
| 의미 | 특정 번호 이전까지 다 받음 | 아직 특정 번호가 비어 있음 |
| 장점 | 중간 ACK 손실 커버 가능    | 손실 조기 감지 가능        |
| 활용 | 기본 TCP ACK               | fast retransmit            |

## 6. rwnd vs cwnd

| 항목      | rwnd               | cwnd               |
| --------- | ------------------ | ------------------ |
| 뜻        | receive window     | congestion window  |
| 결정 주체 | receiver           | sender             |
| 목적      | 버퍼 overflow 방지 | 네트워크 혼잡 방지 |
| 관점      | 수신자 상태        | 네트워크 상태      |

## 7. Slow Start vs Congestion Avoidance

| 항목      | Slow Start             | Congestion Avoidance |
| --------- | ---------------------- | -------------------- |
| 증가 방식 | 빠름(지수적 감각)      | 느림(선형)           |
| 시점      | 연결 초반 / timeout 후 | ssthresh 이후        |
| 목적      | 빠른 초기 탐색         | 조심스러운 미세 탐색 |

## 8. Tahoe vs Reno vs CUBIC

| 항목      | Tahoe             | Reno               | CUBIC                    |
| --------- | ----------------- | ------------------ | ------------------------ |
| 손실 대응 | 보수적            | 더 개선됨          | 고속 환경에 강점         |
| 특징      | timeout 성격 강함 | fast recovery 활용 | Wmax 기준 cubic 증가     |
| 감각      | 안정성 우선       | 성능 균형          | 현대 고속 서버 환경 적합 |

## 9. 2-way handshake vs 3-way handshake

| 항목                | 2-way  | 3-way            |
| ------------------- | ------ | ---------------- |
| 상태 확인           | 불충분 | 양방향 확인 가능 |
| half-open 위험      | 큼     | 줄어듦           |
| duplicate data 문제 | 위험   | 더 안전          |
| 실제 TCP 사용       | X      | O                |

## 10. TCP 기반 HTTP/2 vs QUIC 기반 HTTP/3

| 항목         | HTTP/2 over TCP  | HTTP/3 over QUIC |
| ------------ | ---------------- | ---------------- |
| 기반         | TCP + TLS        | QUIC over UDP    |
| 핸드셰이크   | 상대적으로 길다  | 더 빠를 수 있음  |
| HOL blocking | TCP 레벨 영향 큼 | stream 단위 완화 |
| 유연성       | 낮음             | 높음             |

---

# 마지막 체크리스트

- [ ] 전송 계층이 process-to-process 통신이라는 말 설명 가능
- [ ] UDP / TCP 차이를 기능 기준으로 비교 가능
- [ ] multiplexing / demultiplexing 정의 가능
- [ ] UDP checksum이 완전한 신뢰성 보장이 아니라는 점 설명 가능
- [ ] rdt1.0 ~ rdt3.0 발전 이유 설명 가능
- [ ] stop-and-wait가 왜 비효율적인지 설명 가능
- [ ] GBN과 SR 차이를 재전송 방식 중심으로 설명 가능
- [ ] TCP sequence number와 ACK number를 바이트 기준으로 설명 가능
- [ ] EstimatedRTT / DevRTT / TimeoutInterval 공식 의미 설명 가능
- [ ] duplicate ACK와 fast retransmit 관계 설명 가능
- [ ] rwnd와 flow control 설명 가능
- [ ] 2-way handshake의 문제와 3-way handshake 필요성 설명 가능
- [ ] congestion control과 flow control 차이 설명 가능
- [ ] AIMD, slow start, ssthresh, congestion avoidance 설명 가능
- [ ] cwnd와 rwnd 차이 설명 가능
- [ ] TCP fairness의 이상과 현실 차이 설명 가능
- [ ] QUIC / HTTP3가 왜 등장했는지 설명 가능

---

# 초압축 1페이지 암기 버전

- 전송 계층 = 프로세스 대 프로세스 통신
- UDP = 빠르고 단순, 대신 신뢰성 없음
- TCP = 연결형, 신뢰적, 순서 보장, 바이트 스트림
- 디멀티플렉싱 기준
  - UDP: dest port
  - TCP: 4-tuple
- RDT 핵심 도구 = checksum + ACK + seq + timer + retransmission
- stop-and-wait는 비효율, pipelining이 해결 방향
- GBN = 뒤를 몰아서 재전송
- SR = 필요한 것만 재전송
- TCP ACK = 다음 기대 바이트 번호
- Timeout = EstimatedRTT + 4DevRTT
- duplicate ACK 3개 = fast retransmit
- rwnd = receiver 보호
- cwnd = network 보호
- Flow Control ≠ Congestion Control
- 3-way handshake는 상태를 양방향 확인하기 위해 필요
- AIMD = 조금씩 증가, 손실 시 크게 감소
- slow start = 초반 지수 증가
- ssthresh 이후는 congestion avoidance
- QUIC = UDP 위에서 신뢰성/보안/혼잡 제어를 유연하게 처리

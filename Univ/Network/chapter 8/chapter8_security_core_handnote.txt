# Chapter 8 Security 핵심 전사 노트

> 목적: 바로 필기 + 시험 직전 복습  
> 형식: 핵심어 / 화살표 / 표 / 암기 문장 중심  
> 기준 자료: `Chapter_8_v80_260602_094606.pdf`, `network_note_prompt_template_core_v3.md`  
> 주의: 교안 원문을 대체하는 자료가 아니라, 학습용 핵심 정리 노트

---

## 0. 먼저 외울 핵심 문장

- 네트워크 보안의 목표 = 기밀성 + 인증 + 무결성 + 접근성/가용성
- 기밀성 = 의도된 수신자만 메시지 내용을 이해
- 인증 = 상대가 진짜 그 상대인지 확인
- 무결성 = 전송 중 메시지가 바뀌지 않았음을 확인
- 공격자는 도청, 삽입, 위조, 세션 탈취, 서비스 거부를 할 수 있음
- 대칭키 = 같은 키로 암호화/복호화, 빠르지만 키 공유가 문제
- 공개키 = 공개키로 암호화, 개인키로 복호화, 키 공유 문제 완화
- RSA 보안성 = 큰 수의 소인수분해가 어렵다는 점에 기반
- 실무에서는 공개키로 세션키를 교환하고, 데이터는 대칭키로 암호화
- Nonce = 한 번만 쓰는 값, 재전송 공격 방지에 사용
- 디지털 서명 = 개인키로 메시지 다이제스트에 서명
- 해시 = 긴 메시지를 고정 길이 지문으로 압축
- CA = 공개키와 신원을 묶어 인증서로 보증하는 기관
- 보안 이메일 = 대칭키 + 공개키 + 해시 + 서명을 조합
- TLS = 전송 계층 위에서 HTTPS 보안을 제공하는 프로토콜
- TLS의 핵심 = 핸드셰이크 → 키 생성 → 레코드 암호화 → 안전한 종료
- TLS 1.3 0-RTT = 빠르지만 replay attack 주의
- IPsec = IP 데이터그램 단위의 암호화/인증/무결성 제공
- AH = 인증+무결성, ESP = 인증+무결성+기밀성
- IPsec transport mode = payload 보호, tunnel mode = 원래 IP datagram 전체 보호
- SA = IPsec에서 보안 처리 방식과 키를 저장한 단방향 연결 상태
- WiFi 보안 = AP 연결 + 네트워크 인증 + 세션키 생성 + 암호화 통신
- 방화벽 = 내부망과 외부 인터넷 사이에서 패킷 통과 여부 결정
- Stateful firewall = TCP 연결 상태까지 보고 판단
- IDS = 패킷 내용과 여러 패킷 간 상관관계를 보고 침입 탐지

---

## 1. 네트워크 보안의 목표와 공격 모델 (p2 ~ p8)

- 핵심: 보안은 단순 암호화가 아니라 `기밀성 / 인증 / 무결성 / 가용성`을 함께 다룸
- 구조: Alice/Bob = 정상 통신 주체 → Trudy = 공격자
- 암기: `보안 = 숨기고 + 확인하고 + 안 바뀌게 하고 + 서비스 유지`
- 주의: 암호화만 해도 인증·무결성이 자동 보장되는 것은 아님
- 시험: 보안 목표 4개와 공격자 행위를 구분해서 설명 가능해야 함

| 목표 | 의미 | 대표 기술 |
|---|---|---|
| Confidentiality | 내용 노출 방지 | 암호화 |
| Authentication | 상대 신원 확인 | 인증서, nonce, 서명 |
| Message Integrity | 변조 탐지 | Hash, MAC, HMAC |
| Access / Availability | 서비스 접근·사용 가능 | 방화벽, IDS, DoS 방어 |

```text
[Alice] ─── message ───> [Bob]
    \                    /
     \__ [Trudy: 도청/삽입/위조/탈취/DoS]
```

- 공격자 가능 행위
  - eavesdrop = 메시지 도청
  - insert = 연결 중 메시지 삽입
  - impersonation/spoofing = 송신자 주소·신원 위조
  - hijacking = 진행 중인 연결 탈취
  - denial of service = 자원 고갈로 서비스 사용 방해

---

## 2. 암호학 기본 언어와 공격 방식 (p9 ~ p10)

- 핵심: 암호화는 평문을 키와 알고리즘으로 암호문으로 바꾸는 과정
- 구조: plaintext `m` → encryption key → ciphertext → decryption key → plaintext
- 암기: `암호화 = 못 읽게 만들기`, `복호화 = 다시 읽게 만들기`
- 주의: 키가 노출되면 알고리즘이 좋아도 보안이 무너질 수 있음
- 시험: ciphertext-only / known-plaintext / chosen-plaintext 공격 차이 구분

```text
m = 평문
KA(m) = KA 키로 암호화한 암호문
m = KB(KA(m)) = 복호화 후 원래 메시지
```

| 공격 방식 | 공격자가 가진 것 | 핵심 위험 |
|---|---|---|
| Ciphertext-only | 암호문만 있음 | brute force, 통계 분석 |
| Known-plaintext | 평문-암호문 쌍 일부 | 대응 관계 추정 |
| Chosen-plaintext | 원하는 평문을 암호화 가능 | 알고리즘/키 패턴 분석 쉬움 |

---

## 3. 대칭키 암호화: Substitution, DES, AES (p11 ~ p15)

- 핵심: 송신자와 수신자가 같은 비밀키 `KS`를 공유
- 구조: Alice/Bob이 같은 키 사용 → 빠른 암호화 가능 → 키 공유 문제가 남음
- 암기: `대칭키 = 같은 열쇠 하나를 둘이 같이 씀`
- 주의: 처음 만나는 두 주체가 안전하게 키를 공유하는 문제가 큼
- 시험: 대칭키의 장점과 한계를 공개키와 비교

```text
[Alice] -- KS(m) --> [Bob]
암호화 키 = KS
복호화 키 = KS
```

### 3-1. 단순 치환 암호

- substitution cipher = 글자 하나를 다른 글자로 대체
- monoalphabetic cipher = 알파벳마다 고정 치환표 사용
- 한계: 통계 분석에 취약

### 3-2. DES

- DES = 오래된 대칭키 표준
- 56-bit key, 64-bit block
- brute force에 취약해짐
- 3DES = DES를 3번 적용하여 강화

### 3-3. AES

- AES = DES를 대체한 대칭키 표준
- 128-bit block 처리
- key 길이 = 128 / 192 / 256 bit
- 암기: `AES = 실무형 강한 대칭키 암호`

---

## 4. 공개키 암호화와 RSA (p16 ~ p28)

- 핵심: 공개키는 모두에게 공개, 개인키는 수신자만 보유
- 구조: 공개키로 암호화 → 개인키로 복호화
- 암기: `공개키 = 자물쇠`, `개인키 = 열쇠`
- 주의: 공개키가 진짜 그 사람의 것인지 검증이 필요함 → CA로 연결
- 시험: RSA 키 생성 흐름, 암호화/복호화 수식, 세션키 사용 이유

```text
Bob의 공개키: K_B+
Bob의 개인키: K_B-

Alice: c = K_B+(m)
Bob:   m = K_B-(c)
```

### 4-1. RSA 키 생성 흐름

```text
1) 큰 소수 p, q 선택
2) n = p × q
3) z = (p-1)(q-1)
4) e 선택: e와 z가 서로소
5) d 선택: e × d mod z = 1
6) 공개키 = (n, e)
7) 개인키 = (n, d)
```

수식:
```text
암호화: c = m^e mod n
복호화: m = c^d mod n
```

의미:
```text
m = 메시지를 숫자로 표현한 값
c = 암호문 숫자
n = p × q
```

암기:
```text
RSA = 큰 수 n을 p, q로 다시 쪼개기 어렵다는 점을 이용
```

### 4-2. RSA의 중요한 성질

```text
K_B-(K_B+(m)) = m
K_B+(K_B-(m)) = m
```

- 공개키 먼저 → 개인키 나중 = 기밀성
- 개인키 먼저 → 공개키 나중 = 서명 검증에 활용

### 4-3. 실무에서 RSA만 쓰지 않는 이유

- RSA = 계산 비용 큼
- 대칭키 = 빠름
- 그래서 실무 구조:

```text
1) RSA/공개키로 세션키 KS 교환
2) 이후 실제 데이터는 KS로 대칭키 암호화
```

- 암기: `공개키는 키 교환용, 대칭키는 대량 데이터 암호화용`

---

## 5. 인증 프로토콜과 Nonce (p30 ~ p40)

- 핵심: 인증은 “상대가 진짜인지” 증명하는 과정
- 구조: 단순 선언 → IP 기반 → 비밀번호 → 암호화 비밀번호 → nonce → 공개키 인증
- 암기: `인증은 말로 믿는 게 아니라, 상대만 할 수 있는 계산을 확인`
- 주의: 암호화된 비밀번호도 replay attack에는 취약할 수 있음
- 시험: ap1.0 ~ ap5.0의 실패 이유를 설명

| 프로토콜 | 방식 | 문제점 |
|---|---|---|
| ap1.0 | “나는 Alice”라고 말함 | Trudy가 그대로 말하면 됨 |
| ap2.0 | IP 주소 포함 | IP spoofing 가능 |
| ap3.0 | 비밀번호 전송 | 도청 후 재전송 가능 |
| ap3.0 수정 | 암호화된 비밀번호 전송 | 암호문 자체 replay 가능 |
| ap4.0 | nonce + 공유키 | 공유키 사전 보유 필요 |
| ap5.0 | nonce + 공개키 | 공개키 자체가 가짜면 MITM 가능 |

### 5-1. Nonce 기반 인증

- nonce = 한 번만 쓰는 임시 숫자 `R`
- 목적: “지금 살아 있는 상대”인지 확인

```text
Bob → Alice: R
Alice → Bob: K_AB(R)
Bob: 공유키로 복호화/검증
```

- 암기: `Nonce = 재방송 방지용 1회용 질문`

### 5-2. Man-in-the-Middle Attack

- 핵심: Trudy가 Alice에게는 Bob처럼, Bob에게는 Alice처럼 행동
- 원인: Bob이 받은 공개키가 진짜 Alice의 공개키인지 검증하지 못함

```text
Bob ── public key 요청 ──> Trudy ── public key 요청 ──> Alice
Bob <── Trudy의 공개키 ─── Trudy <── Alice의 공개키 ───── Alice

Bob은 Trudy의 키를 Alice의 키로 착각
```

- 해결 방향: 공개키 인증서 + CA 필요

---

## 6. 디지털 서명, 해시, 메시지 다이제스트, CA (p42 ~ p51)

- 핵심: 무결성과 인증은 해시 + 개인키 서명으로 제공
- 구조: 메시지 → 해시 → 개인키로 서명 → 공개키로 검증
- 암기: `서명 = 개인키로 찍는 도장`, `검증 = 공개키로 확인`
- 주의: 메시지 전체를 공개키로 암호화하면 비용이 큼 → 해시값에 서명
- 시험: 디지털 서명과 암호화의 목적 차이를 구분

### 6-1. 디지털 서명

```text
Bob이 서명:
서명값 = K_B-(m)

Alice가 검증:
K_B+(K_B-(m)) = m ?
```

- 제공 기능
  - 작성자 확인
  - 위조 어려움
  - 부인 방지(non-repudiation)

### 6-2. 메시지 다이제스트

- hash function = 큰 메시지를 고정 길이 지문으로 변환
- 속성:
  - many-to-one
  - fixed-size digest
  - digest만 보고 원래 메시지 찾기 어려움
  - 같은 digest를 갖는 다른 메시지 찾기 어려워야 함

```text
m ── H() ──> H(m)
긴 메시지      고정 길이 지문
```

- 주의: Internet checksum은 암호학적 해시로 부적합
  - 고정 길이 값은 만들지만 충돌을 만들기 쉬움

### 6-3. 실제 디지털 서명 구조

```text
Bob:
1) H(m) 계산
2) K_B-(H(m)) 생성
3) m + 서명값 전송

Alice:
1) 받은 m으로 H(m) 계산
2) K_B+(서명값) 계산
3) 두 값 비교
```

- 암기: `긴 메시지는 직접 서명 X, 해시를 서명 O`

### 6-4. CA와 인증서

- 문제: 공개키가 진짜 Bob의 것인지 확인 필요
- CA = Certification Authority
- 역할: 신원 + 공개키를 묶어 인증서 생성

```text
Bob의 신원 + Bob 공개키 ── CA 개인키로 서명 ──> Bob 인증서
Alice는 CA 공개키로 인증서 검증
```

- 암기: `CA = 공개키 신분증 발급 기관`

---

## 7. 보안 이메일 (p53 ~ p56)

- 핵심: 이메일 보안은 기밀성, 무결성, 인증을 조합해야 함
- 구조: 메시지는 대칭키로 암호화, 대칭키는 수신자 공개키로 암호화
- 암기: `메일 본문은 빠른 대칭키, 대칭키 전달은 공개키`
- 주의: 서명만 하면 내용은 평문일 수 있음 / 암호화만 하면 작성자 인증은 별도 필요
- 시험: Alice가 사용하는 키 3개를 구분

### 7-1. 기밀성만 제공

```text
Alice:
1) 임시 대칭키 KS 생성
2) 메일 m을 KS로 암호화 → KS(m)
3) KS를 Bob 공개키로 암호화 → K_B+(KS)
4) 둘 다 Bob에게 전송

Bob:
1) 개인키 K_B-로 KS 복구
2) KS로 메일 m 복호화
```

### 7-2. 무결성 + 인증 제공

```text
Alice:
1) H(m) 계산
2) 자신의 개인키 K_A-로 H(m)에 서명
3) m + K_A-(H(m)) 전송

Bob:
1) m으로 H(m) 재계산
2) Alice 공개키 K_A+로 서명 복호화
3) 두 digest 비교
```

### 7-3. 기밀성 + 무결성 + 인증 모두 제공

```text
[메시지 m]
   ├─ H(m) → Alice 개인키로 서명 → 인증/무결성
   └─ KS로 암호화 → 기밀성

[KS]
   └─ Bob 공개키로 암호화 → 안전한 키 전달
```

- Alice가 쓰는 키:
  - `K_A-` = Alice 개인키, 서명용
  - `K_B+` = Bob 공개키, 세션키 보호용
  - `KS` = 메시지 암호화용 대칭키

---

## 8. TLS: TCP 연결 보안 (p58 ~ p69)

- 핵심: TLS는 전송 계층 위에서 동작하며 HTTPS 보안을 제공
- 구조: TCP 연결 → TLS handshake → 키 생성 → record 단위 암호화 → 안전 종료
- 암기: `TLS = 대칭키 + 해시 + 공개키 인증을 TCP 위에 얹은 것`
- 주의: TLS는 IP 자체가 아니라 애플리케이션이 사용하는 안전한 통신 채널 제공
- 시험: TLS가 제공하는 3가지 기능과 handshake 과정을 설명

### 8-1. TLS가 제공하는 것

| 기능 | 구현 방식 |
|---|---|
| Confidentiality | 대칭키 암호화 |
| Integrity | 암호학적 해시 / MAC |
| Authentication | 공개키 인증서 |

- HTTPS = HTTP + TLS
- 기본 포트 = 443
- SSL은 과거 방식, 현재는 TLS 사용

### 8-2. TLS에 필요한 구성

```text
1) Handshake
   - 인증서 확인
   - 공유 secret 생성

2) Key derivation
   - master secret에서 여러 키 생성

3) Data transfer
   - TCP byte stream을 record 단위로 나누어 보호

4) Connection closure
   - 종료 메시지도 보호
```

### 8-3. TLS의 여러 키

- 같은 키를 여러 기능에 재사용하면 위험
- 방향별/목적별 키 분리

| 키 | 의미 |
|---|---|
| Kc | client → server 암호화 키 |
| Mc | client → server MAC 키 |
| Ks | server → client 암호화 키 |
| Ms | server → client MAC 키 |

- 암기: `TLS 키 = 방향별 + 기능별로 나눔`

### 8-4. TLS record

```text
TCP는 byte stream
→ 그대로 암호화하면 중간 검증 어려움
→ record 단위로 자름
→ 각 record에 MAC 포함
```

```text
[ data | MAC | length | type ] → 대칭키로 암호화 → TCP로 전송
```

- reordering/replay 방지:
  - TLS sequence number를 MAC 계산에 포함
  - nonce 사용

### 8-5. 안전한 연결 종료

- truncation attack = 공격자가 TCP close를 위조하여 일부 데이터가 없는 것처럼 보이게 함
- 해결:
  - record type에 `data` / `close` 구분
  - MAC 계산에 data + type + sequence number 포함

### 8-6. TLS 1.3 handshake

```text
1-RTT handshake:
ClientHello → ServerHello + Certificate → Client verifies → Application data
```

- ClientHello:
  - 지원 cipher suite
  - DH key agreement 정보
- ServerHello:
  - 선택된 cipher suite
  - DH parameter
  - 서버 인증서
- Client:
  - 인증서 확인
  - 키 생성
  - HTTPS 요청 가능

### 8-7. TLS 1.3 0-RTT

- 이전 연결 재개 시 첫 메시지에 application data 포함 가능
- 장점: 빠름
- 단점: replay attack 취약
- 주의: 서버 상태를 바꾸는 요청에는 위험할 수 있음

---

## 9. IPsec: 네트워크 계층 보안 (p70 ~ p82)

- 핵심: IPsec은 IP 데이터그램 단위로 암호화/인증/무결성을 제공
- 구조: 정책 확인(SPD) → SA 선택(SAD) → AH/ESP 적용 → 전송
- 암기: `TLS는 연결 위, IPsec은 IP 패킷 자체 보호`
- 주의: IP는 원래 connectionless지만, IPsec은 SA라는 상태를 가짐
- 시험: transport mode/tunnel mode, AH/ESP, SA/SPI/SPD/SAD 구분

### 9-1. IPsec의 두 모드

| 모드 | 보호 범위 | 특징 |
|---|---|---|
| Transport mode | IP payload | 원래 IP header는 유지 |
| Tunnel mode | 원래 IP datagram 전체 | 새 IP header로 감싸 터널링 |

```text
Transport mode:
[IP header][protected payload]

Tunnel mode:
[new IP header][protected original IP datagram]
```

- 암기: `터널 모드 = 원래 패킷을 통째로 상자에 넣고 새 주소표를 붙임`

### 9-2. AH와 ESP

| 프로토콜 | 제공 기능 | 기밀성 |
|---|---|---|
| AH | source authentication + integrity | X |
| ESP | source authentication + integrity + confidentiality | O |

- ESP가 더 널리 사용됨
- AH = “누가 보냈고 안 바뀌었나”
- ESP = “누가 보냈고 안 바뀌었고 내용도 숨겼나”

### 9-3. Security Association, SA

- SA = 송신자 → 수신자 방향의 보안 상태
- 단방향이므로 양방향 통신에는 SA 2개 필요
- SA에 저장되는 정보:
  - SPI
  - 출발/도착 인터페이스
  - 암호화 알고리즘
  - 암호화 키
  - 무결성 검사 알고리즘
  - 인증 키

```text
SA = 어떤 알고리즘/키/SPI로 이 방향 트래픽을 보호할지 적어둔 표
```

### 9-4. SPI와 IPsec datagram

- SPI = Security Parameter Index
- 수신자가 어떤 SA를 써야 하는지 찾는 식별자
- sequence number = replay attack 방지

```text
[new IP header]
[ESP header: SPI, Seq#]
[encrypted original datagram + ESP trailer]
[ESP auth: MAC]
```

### 9-5. ESP tunnel mode 처리 흐름

```text
R1:
1) 원래 datagram 뒤에 ESP trailer 추가
2) SA에 지정된 키/알고리즘으로 암호화
3) 앞에 ESP header 추가
4) MAC 생성 후 ESP auth 추가
5) 새 IP header 추가
6) tunnel endpoint로 전송
```

### 9-6. SPD와 SAD

| DB | 의미 | 질문 |
|---|---|---|
| SPD | Security Policy Database | 이 datagram에 IPsec을 적용할까? |
| SAD | Security Association Database | 어떤 SA/키/알고리즘으로 처리할까? |

- 암기: `SPD = What`, `SAD = How`

### 9-7. IKE

- IKE = Internet Key Exchange
- 수동으로 SA/키 설정은 대규모 VPN에서 비현실적
- IKE가 알고리즘, 키, SPI 등을 협상

| 방식 | 출발점 | 특징 |
|---|---|---|
| PSK | 미리 공유한 비밀값 | 양쪽이 같은 secret 보유 |
| PKI | 공개키/개인키/인증서 | 인증서 기반 인증 |

### 9-8. IKE Phase

```text
Phase 1: IKE SA 생성
Phase 2: IPsec SA 쌍 협상
```

- main mode = 더 유연, identity 보호
- aggressive mode = 메시지 수 적음

---

## 10. 무선·모바일 보안: 802.11, WPA3, 4G/5G (p83 ~ p99)

- 핵심: 무선 보안은 접속 자체보다 인증과 세션키 생성이 핵심
- 구조: AP 발견 → 인증 방식 선택 → 상호 인증 → 세션키 생성 → 암호화 통신
- 암기: `WiFi 보안 = 연결 + 인증 + 키 생성 + 암호화`
- 주의: AP와 메시지를 주고받는다고 해서 이미 인증된 것은 아님
- 시험: 802.11 인증 단계와 4G/5G 차이를 구분

### 10-1. 802.11 인증·암호화 단계

```text
1) Discovery
   AP가 보안 기능 광고
   Mobile이 원하는 인증/암호화 방식 요청

2) Mutual authentication + key derivation
   AS와 Mobile이 공유 secret + nonce + hash로 서로 인증
   session key 생성

3) Shared symmetric key distribution
   AS가 AP에게 session key 전달

4) Encrypted communication
   Mobile ↔ AP 구간 암호화 통신
```

```text
[Mobile] ⇄ [AP] ⇄ [AS]
 단말       접속점   인증 서버
```

### 10-2. WPA3 handshake 핵심

```text
AS → Mobile: Nonce_AS
Mobile:
  - Nonce_M 생성
  - initial shared secret + Nonce_AS + Nonce_M으로 session key 생성
  - HMAC 값 전송
AS:
  - 같은 방식으로 session key 생성
```

- nonce = relay/replay 공격 방지
- HMAC = 메시지 무결성 및 인증 확인
- session key = 이후 AES 등으로 암호화 통신

### 10-3. EAP

- EAP = Extensible Authentication Protocol
- Mobile ↔ AS 사이의 request/response 인증 프레임워크
- WiFi 구간에서는 EAPoL 사용
- AP ↔ AS 구간에서는 RADIUS/UDP/IP 사용 가능

### 10-4. 4G LTE 인증·암호화

- 구성:
  - Mobile
  - Base Station(BS)
  - Visited network의 MME
  - Home network의 HSS
- SIM = 단말의 글로벌 신원과 공유키 포함
- HSS = 최종 인증자 역할

```text
Mobile → BS → MME → HSS
attach / IMSI / visited network info 전달
```

- 흐름:
```text
1) Mobile이 attach 메시지 전송
2) MME가 HSS에 인증 요청
3) HSS가 auth token, expected response, key 생성
4) Mobile이 resM 계산 후 MME에 전송
5) MME가 resM과 xresHSS 비교
6) 일치하면 인증 성공, BS와 Mobile이 암호화 키 사용
```

### 10-5. 4G와 5G 차이

| 구분 | 4G | 5G |
|---|---|---|
| 인증 결정 | 방문망 MME가 결정 | 홈 네트워크가 결정 |
| 키 전제 | 미리 공유된 키 중심 | IoT 등에서 사전 공유키 부담 완화 |
| IMSI | 평문 전송 가능 | 공개키 암호로 IMSI 보호 |

- 암기: `5G = 홈망 중심 인증 + IMSI 보호 강화`

---

## 11. 방화벽과 접근 제어 (p100 ~ p110)

- 핵심: 방화벽은 내부망과 외부 인터넷 사이에서 허용/차단 정책 적용
- 구조: packet header 검사 → rule과 비교 → allow/drop
- 암기: `방화벽 = 네트워크 출입문 + 규칙표`
- 주의: 방화벽이 모든 공격을 막는 것은 아님
- 시험: stateless / stateful / application gateway 차이를 구분

### 11-1. 방화벽 목적

- DoS 방어
  - 예: SYN flooding으로 가짜 TCP 연결 대량 생성
- 내부 데이터 불법 접근/수정 방지
- 허가된 사용자/호스트만 내부망 접근 허용

### 11-2. Stateless Packet Filter

- 패킷 하나하나 독립적으로 판단
- 확인 필드:
  - source IP
  - destination IP
  - TCP/UDP source port
  - TCP/UDP destination port
  - ICMP type
  - TCP SYN/ACK bit

```text
패킷 도착 → 헤더 필드 확인 → ACL 위에서부터 검사 → allow/drop
```

- 예:
  - inbound TCP `ACK=0` 차단 → 외부에서 내부로 새 TCP 연결 시작 방지
  - port 80 outbound 차단 → 외부 웹 접근 차단

### 11-3. ACL

- ACL = Access Control List
- `(action, condition)` 쌍의 rule table
- 위에서 아래로 적용
- 마지막은 보통 `deny all`

```text
allow 조건1
allow 조건2
deny all
```

- 암기: `ACL = 위에서부터 읽는 출입 규칙표`

### 11-4. Stateful Packet Filter

- stateless의 한계: 맥락 없이 ACK 패킷 등을 허용할 수 있음
- stateful = TCP 연결 상태를 추적
- 확인 대상:
  - SYN으로 연결 시작했는가?
  - FIN으로 종료됐는가?
  - 비활성 timeout이 지났는가?

```text
패킷 도착 → ACL 검사 → connection table 확인 → allow/drop
```

- 암기: `Stateful = 대화 흐름을 기억하는 방화벽`

### 11-5. Application Gateway

- 애플리케이션 데이터까지 검사
- 예: telnet은 반드시 gateway를 통해서만 허용
- gateway가 내부 사용자와 외부 서버 사이를 중계

```text
[Internal Host] ⇄ [Application Gateway] ⇄ [Remote Host]
```

- 장점: 애플리케이션 수준 정책 가능
- 단점: 앱마다 gateway 필요 가능, 클라이언트 설정 필요

### 11-6. 방화벽의 한계

- IP spoofing은 완전 판별 어려움
- UDP 정책은 all-or-nothing이 되기 쉬움
- 앱별 특수 처리가 많으면 복잡해짐
- 외부와의 통신 자유도와 보안 수준은 trade-off
- 매우 보호된 사이트도 공격을 받을 수 있음

---

## 12. IDS: Intrusion Detection System (p111 ~ p113)

- 핵심: IDS는 패킷 내용과 여러 패킷의 관계를 보고 침입을 탐지
- 구조: packet capture → deep inspection → signature/correlation 분석 → alert
- 암기: `방화벽 = 막기`, `IDS = 수상한 행동 찾기`
- 주의: IDS는 탐지 중심이며, 배치 위치와 탐지 방식이 중요
- 시험: packet filter와 IDS의 차이를 설명

### 12-1. Packet Filtering의 한계

- 주로 TCP/IP header 기반 판단
- 세션 간 상관관계 분석 약함
- payload 내부 공격 문자열 확인 어려움

### 12-2. IDS가 보는 것

- Deep Packet Inspection
  - packet payload 내용 검사
  - 바이러스/공격 문자열 database와 비교
- 여러 packet 간 상관관계
  - port scanning
  - network mapping
  - DoS attack pattern

### 12-3. IDS 배치

```text
Internet → Firewall → DMZ(Web/FTP/DNS) → Internal Network
              │              │                  │
             IDS            IDS                IDS
```

- 위치별 목적:
  - 외부 경계 = 외부 공격 탐지
  - DMZ = 공개 서버 공격 탐지
  - 내부망 = 침투 후 lateral movement 탐지

---

## 마지막 비교 정리

| 비교 | A | B | 핵심 차이 |
|---|---|---|---|
| 기밀성 vs 무결성 | 기밀성: 내용 숨김 | 무결성: 변조 탐지 | 숨기는 것과 바뀌었는지 확인은 다름 |
| 인증 vs 무결성 | 인증: 누가 보냈나 | 무결성: 안 바뀌었나 | 인증은 신원, 무결성은 내용 |
| 대칭키 vs 공개키 | 같은 키 공유, 빠름 | 공개키/개인키, 느림 | 실무는 둘을 조합 |
| DES vs AES | 오래된 56-bit 대칭키 | 현대적 대칭키 표준 | AES가 더 강하고 일반적 |
| 공개키 암호화 vs 디지털 서명 | 수신자 공개키로 암호화 | 송신자 개인키로 서명 | 목적이 기밀성 vs 인증/부인방지 |
| Hash vs Encryption | 고정 길이 지문 생성 | 복호화 가능한 암호문 생성 | 해시는 원문 복구 목적 아님 |
| Checksum vs Crypto Hash | 오류 검출용 | 보안 무결성용 | checksum은 보안 해시로 약함 |
| Nonce vs Session Key | 1회용 값 | 통신 동안 쓰는 대칭키 | nonce는 재전송 방지, session key는 암호화 |
| Replay vs MITM | 이전 메시지 재전송 | 중간자 위장 | 둘 다 인증 설계에서 중요 |
| CA vs Certificate | 인증서를 발급/서명하는 기관 | 공개키+신원+CA 서명 | CA가 공개키 신뢰를 보증 |
| TLS vs IPsec | 전송 계층 위 보안 | 네트워크 계층 보안 | HTTPS는 TLS, VPN은 IPsec 활용 가능 |
| TLS 1-RTT vs 0-RTT | 인증 후 데이터 | 첫 메시지부터 데이터 | 0-RTT는 replay 주의 |
| IPsec transport vs tunnel | payload만 보호 | 원래 datagram 전체 보호 | tunnel은 새 IP header로 캡슐화 |
| AH vs ESP | 인증+무결성 | 인증+무결성+기밀성 | ESP가 더 널리 쓰임 |
| SPD vs SAD | 무엇을 할지 정책 | 어떻게 처리할지 상태 | SPD=What, SAD=How |
| PSK vs PKI | 사전 공유 비밀 | 인증서/공개키 기반 | 규모 커질수록 PKI 유리 |
| WiFi AS vs 4G HSS | WiFi 인증 서버 | 이동통신 홈망 인증 서버 | 둘 다 최종 인증자 역할 |
| Stateless firewall vs Stateful firewall | 패킷 단위 판단 | 연결 상태 추적 | stateful이 맥락을 봄 |
| Firewall vs IDS | 차단 중심 | 탐지 중심 | 방화벽은 문지기, IDS는 감시자 |
| Packet filter vs Application gateway | 헤더 필드 중심 | 앱 데이터까지 검사 | gateway는 더 깊지만 복잡 |

---

## 최종 체크리스트

- [ ] 네트워크 보안 목표 4가지를 설명할 수 있다.
- [ ] Trudy가 할 수 있는 공격 유형을 예시로 말할 수 있다.
- [ ] 대칭키와 공개키의 차이를 설명할 수 있다.
- [ ] RSA 키 생성 흐름과 `c = m^e mod n`, `m = c^d mod n` 의미를 말할 수 있다.
- [ ] 공개키를 데이터 전체 암호화보다 세션키 교환에 많이 쓰는 이유를 설명할 수 있다.
- [ ] replay attack과 nonce의 관계를 설명할 수 있다.
- [ ] MITM 공격이 왜 공개키 검증 문제인지 설명할 수 있다.
- [ ] 디지털 서명과 해시의 역할을 분리해서 설명할 수 있다.
- [ ] CA와 인증서가 왜 필요한지 설명할 수 있다.
- [ ] 보안 이메일에서 Alice가 사용하는 세 가지 키를 구분할 수 있다.
- [ ] TLS handshake, key derivation, record, close 흐름을 말할 수 있다.
- [ ] TLS 1.3 1-RTT와 0-RTT 차이를 설명할 수 있다.
- [ ] IPsec transport mode와 tunnel mode를 그림으로 설명할 수 있다.
- [ ] AH와 ESP 차이를 말할 수 있다.
- [ ] SA, SPI, SPD, SAD를 구분할 수 있다.
- [ ] IKE phase 1/2의 역할을 설명할 수 있다.
- [ ] 802.11 인증·암호화 단계를 순서대로 말할 수 있다.
- [ ] WPA3 handshake에서 nonce와 HMAC의 역할을 설명할 수 있다.
- [ ] 4G LTE 인증 흐름에서 MME와 HSS 역할을 설명할 수 있다.
- [ ] 4G와 5G 보안 차이를 표 없이 설명할 수 있다.
- [ ] stateless/stateful/application gateway 방화벽을 구분할 수 있다.
- [ ] ACL이 위에서 아래로 적용된다는 점을 설명할 수 있다.
- [ ] IDS가 packet filter보다 더 깊게 보는 항목을 설명할 수 있다.

---

## 시험 직전 1분 압축

```text
보안 목표: 기밀성 / 인증 / 무결성 / 가용성
암호화: 대칭키는 빠름, 공개키는 키 교환·서명에 유리
RSA: 공개키(n,e), 개인키(n,d), 큰 수 소인수분해 어려움
인증: nonce로 replay 방지, CA로 공개키 신뢰 보장
서명: H(m)을 개인키로 서명, 공개키로 검증
메일: KS로 본문 암호화, K_B+로 KS 보호, K_A-로 서명
TLS: handshake → key derivation → record 암호화 → close 보호
IPsec: AH/ESP, transport/tunnel, SA/SPI/SPD/SAD/IKE
WiFi/모바일: 인증 서버와 세션키 생성이 핵심
운영 보안: 방화벽은 차단, IDS는 탐지
```

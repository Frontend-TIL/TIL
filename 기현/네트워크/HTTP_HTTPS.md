# HTTP & HTTPS 정리 (TIL)

---

## 📌 HTTP 특징

### 1. 비연결성 (Connectionless)
- 요청 → 응답 후 바로 연결 종료
- 장점: 서버 자원 절약 (CPU, 메모리)
- 단점: 매 요청마다 TCP 연결 → 오버헤드 발생
- 해결: HTTP/1.1 Keep-Alive

---

### 2. 무상태성 (Stateless)
- 서버는 이전 요청 상태를 기억하지 않음
- 확장성 ↑ (Scale-out 유리)
- 단점: 로그인 상태 유지 어려움

👉 해결 방법
- 쿠키 / 세션
- JWT (토큰 기반 인증)

---

### 3. 요청 / 응답 구조
- 클라이언트 → 요청 → 서버 → 응답
- 서버가 먼저 데이터 보내는 구조 ❌
- 단방향 통신

---

### 4. 미디어 독립성
- Content-Type으로 데이터 타입 정의
- JSON, HTML, 이미지 등 모두 전송 가능

---

## 📌 상태 유지 방식

### 1. 쿠키 (Client 저장)
- 브라우저에 저장
- 요청 시 자동 전송

**장점**
- 서버 자원 사용 없음

**단점**
- 보안 취약 (변조 가능)

---

### 2. 세션 (Server 저장)
- 서버가 사용자 정보 저장
- 클라이언트는 Session ID만 쿠키로 전달

**장점**
- 보안 ↑

**단점**
- 서버 부하 ↑
- 서버 확장 시 세션 공유 필요

---

### 3. JWT (토큰 기반)
- 서버는 상태 저장 X (Stateless)
- 클라이언트가 토큰 저장 후 요청마다 전달

**장점**
- 확장성 ↑

**단점**
- 탈취 시 위험

👉 실무 전략
- Access Token (짧은 수명)
- Refresh Token (HttpOnly Cookie)

---

## 📌 HTTP 1.1 vs HTTP 2.0

| 구분 | HTTP/1.1 | HTTP/2 |
|------|----------|--------|
| 통신 방식 | 순차 처리 | 병렬 처리 |
| 데이터 형식 | 텍스트 | 이진 |
| 멀티플렉싱 | ❌ | ✅ |
| 헤더 압축 | ❌ | ✅ |
| 문제 | HOL Blocking | 개선됨 |

---

### 🔥 HTTP/2 핵심 개선
- 멀티플렉싱 → 동시에 여러 요청 처리
- 헤더 압축 (HPACK)
- Server Push
- 하나의 TCP 연결로 처리

👉 목적: **웹 성능 개선**

---

## 📌 HTTPS 동작 과정

👉 HTTP + SSL/TLS (암호화)

### 1. TCP 3-Way Handshake

---

### 2. TLS Handshake

1. Client Hello
   - 지원 암호 방식 전달

2. Server Hello
   - 암호 방식 선택 + 인증서 전달

3. 인증서 검증
   - CA 기반 검증

4. Pre-Master Secret 생성
   - 공개키로 암호화 후 전달

---

### 3. 데이터 통신

- 대칭키 생성
- 이후 데이터는 대칭키로 암호화

---

### 🔥 핵심 구조
- 초기: 비대칭키 (보안)
- 이후: 대칭키 (속도)

---

## 📌 TLS 역할

- 데이터 암호화 (기밀성)
- 데이터 위변조 방지 (무결성)
- 서버 인증 (인증)

---

## 📌 HTTP vs HTTPS

| 항목 | HTTP | HTTPS |
|------|------|-------|
| 암호화 | ❌ | ✅ |
| 보안 | 낮음 | 높음 |
| 인증 | 없음 | 있음 |
| 포트 | 80 | 443 |

---

## 📌 GET vs POST

| 구분 | GET | POST |
|------|-----|------|
| 목적 | 조회 | 생성/처리 |
| 데이터 위치 | URL | Body |
| 멱등성 | O | X |
| 캐싱 | 가능 | 불가능 |
| 보안 | 낮음 | 상대적 높음 |

---

## 📌 REST API

👉 자원을 URI로 표현하고 HTTP Method로 처리

### REST 특징
1. Stateless
2. Cacheable
3. Client-Server 구조
4. Layered System
5. Uniform Interface

---

### RESTful 예시

| 행위 | Method | URI |
|------|--------|-----|
| 조회 | GET | /users/1 |
| 생성 | POST | /users |
| 수정 | PUT | /users/1 |
| 삭제 | DELETE | /users/1 |

---

## 🔥 한줄 정리

- HTTP = 비상태, 비연결
- HTTPS = HTTP + TLS (보안)
- HTTP/2 = 성능 개선 (멀티플렉싱)
- 인증 = 세션 vs JWT

# 경력기술서 — 쿠콘 지원용

> 진현규 | Backend Engineer | jinhyeonkyu@gmail.com
> GitHub: https://github.com/Hyeonqz | Blog: https://hyeonq.tistory.com

---

## (주) 큐뱅 · Server Developer · 2024.01 ~ 현재

핀테크 스타트업으로, QR 기반 결제 중계 플랫폼을 운영합니다.
국내외 간편결제사와 가맹점(POS) 사이의 결제 데이터를 중계·처리하는 B2B 금융 서비스입니다.
서버 개발자로 입사하여 QR 결제 중계 시스템 설계·구현·운영 전 과정을 담당하고 있습니다.

---

### 1. 다기관 금융 API 연동 및 QR 결제 중계 플랫폼 구축

**개요**
알리페이·위챗페이·라인페이·유니온페이 등 국내외 간편결제사 7개사 이상과
가맹점 POS를 연결하는 결제 중계 시스템을 0에서 설계하고 운영했습니다.
각 기관마다 상이한 인증 방식·요청/응답 포맷·재시도 정책을 내부 표준 인터페이스로 통합했습니다.

**역할**
- 연동 대상 금융기관 API Spec 분석 및 내부 표준화 레이어 설계 (단독 담당)
- POS 및 간편결제사 REST API Spec 15개 이상 정의 및 구현
- 외부 기관별 인증·서명 방식 추상화로 신규 결제사 추가 공수 최소화
- C++ DLL 기반 레거시 POS 네이티브 모듈 개발로 이기종 시스템 통합
- Spring Boot & React 기반 결제 테스트 자동화 웹앱 개발 — POS 독립 테스트 환경 구축으로 외부기관 커뮤니케이션 비용 **50% 절감**

**기술스택**
Java · Spring Boot · JPA · MySQL · Redis · Kafka · AWS · Docker

**기술적 개선 사항**

> **Pinpoint APM 모니터링 체계 구축**
> - 배경: 운영 중 API 호출 실패 발생 시 병목 지점 식별 불가 — 장애 대응 지연 구조
> - 해결: Pinpoint APM 도입으로 전체 API 호출 흐름 시각화, 외부 금융기관과의 연동 실패 구간 자동 식별
> - 결과: API 호출 실패율 **90% 감소** — 다수 외부 기관과 연동하는 쿠콘 환경에서 직결되는 역량

> **Kafka Request-Reply 아키텍처 설계 — 분산 환경 응답 정합성 확보**
> - 배경: 분산 환경에서 Kafka Reply 토픽 수신 Consumer 인스턴스 불일치로 타임아웃 빈번 발생
> - 해결: CorrelationId 기반 Reply 라우팅으로 Consumer 간 응답 격리 구현, DLQ 처리 및 재시도 흐름 설계
> - 결과: 타임아웃 **50% → 0%**, 수동 CS 월 **5건 → 2건 (60%↓)** &nbsp;[상세보기](https://www.notion.so/01-Kafka-3345cb02eeeb80d9a786d314378e2138)

> **Redis Cache-Aside + 트랜잭션 경계 최적화 — 조회 성능 개선**
> - 배경: 결제 상태 Polling 120회 기준 DB 직접 조회 360회 발생 — 응답 지연 및 DB 부하
> - 해결: Cache-Aside 패턴 도입, 트랜잭션 커밋 후 캐시 갱신 순서 보장으로 더티 리드 방지
> - 결과: DB 쿼리 **360회 → 0회**, 응답시간 **200ms → 30ms (85%↓)** &nbsp;[상세보기](https://www.notion.so/02-Redis-3345cb02eeeb81fc8f2ece133ba5b68a)

> **Final API 패턴 도입 — 외부 기관 연동 상태 동기화**
> - 배경: POS 결제 완료 후 서버 상태 미동기화로 CS 문의 월 10건 반복
> - 해결: SSE·Webhook·Final API 등 방식을 시장 조사 후 비용·구조 적합성 기준으로 팀 회의 및 사내 PT 발표를 통해 Final API 방식으로 확정. 결제 최종 상태 단방향 확정 및 멱등성 보장 로직 구현
> - 결과: CS 월 **10건 → 0~1건 (95%↓)**, 연간 수동 공수 **40시간 절감** &nbsp;[상세보기](https://www.notion.so/03-CS-95-Final-API-3345cb02eeeb81afb108d6d2517a8d6f)

> **Outbox 패턴 + SFTP 세션 풀링 — 파일 전송 안정성 확보**
> - 배경: SFTP 파일 전송 실패 시 유실 감지 불가 — 파일 유실률 100%, 수동 모니터링 의존
> - 해결: Outbox 패턴으로 전송 이벤트 DB 영속화, SFTP 세션 풀링으로 연결 안정성 확보, 파일 I/O 비동기(`@Async`) 처리
> - 결과: 파일 유실률 **100% → 0%**, 응답시간 **4s → 1s (75%↓)**, 10,000건 무장애 처리, 장애 감지 **30분 이내 자동화** &nbsp;[상세보기](https://www.notion.so/04-Outbox-0-3345cb02eeeb81ecbc74e89dfb547a56)

**종합 성과**
- 일 평균 거래 **50건 → 4,000건 (80배)** 성장 무장애 운영
- 이기종 결제사 7개사 연동을 단일 표준 인터페이스로 통합
- 월 거래액 **30% 확장** (레거시 POS 통합 이후)

---

### 2. PCI DSS 컴플라이언스 달성 — 금융 보안 아키텍처 구축

**개요**
금융당국 PCI DSS 자격 취득을 위한 보안 아키텍처를 구축했습니다.
금융 데이터를 다루는 쿠콘 환경에서 동일하게 요구되는 보안 규정 준수 경험입니다.

**역할 및 구현**
- **HashiCorp Vault 도입** — 기존 application.yml 평문 저장 방식의 암호화 키를 Vault 중앙집중식 관리로 전환
  - Spring Cloud Vault 연동으로 런타임 시크릿 주입, 커스텀 어노테이션으로 암호화·복호화 자동화
  - 키 로테이션 자동화로 보안 운영 효율성 향상
- **Keycloak 기반 백오피스 2FA 구현** — 인증/인가 시스템 강화
- **민감정보 접근 로그 추적 체계 구축** — JPA Auditing + Kafka 활용, 금융 데이터 접근 이력 관리
- 코드 리뷰 체크리스트에 PCI DSS 요구사항 반영
- 보안 취약점 해결: High **5건**, Medium **12건**

**기술스택**
HashiCorp Vault · Spring Cloud Vault · Keycloak · Kafka · Java · Spring Boot

---

### 3. 애플리케이션 아키텍처 개선 — 멀티 모듈 + 헥사고날 전환

**개요**
서비스 확장에 따른 유지보수 효율화를 위해 모놀리식 구조를 멀티 모듈 + 헥사고날 아키텍처로 전환했습니다.

**역할 및 구현**
- 모놀리식 → Gradle 멀티 모듈 아키텍처 전환, 레이어드 → DDD 기반 헥사고날 아키텍처 도입
- 멀티 모듈 전이 의존성 제거로 Gradle 빌드 시간 **55s → 35s** 단축
- Nexus를 활용해 공통 JPA Entity·유틸 라이브러리 분리 — 5개 프로젝트 공통 적용으로 반복 코드 제거
- 이벤트 기반 아키텍처 전환으로 모듈 간 결합도 감소 및 MSA 확장 기반 마련

**기술스택**
Java · Spring Boot · Gradle · Nexus · JPA

---

### 4. 모바일 POS 시스템 구축

**개요**
신규 모바일 POS 서비스 런칭을 위한 서버 환경을 처음부터 구축했습니다.

**역할 및 구현**
- CI/CD 파이프라인 설계 및 구축 (Jenkins · Docker · AWS)
- 헥사고날 아키텍처 기반 백엔드 API 구조 표준화
- React Native 기반 크로스 플랫폼 앱 개발 참여

**기술스택**
Java · Spring Boot · Jenkins · Docker · AWS · React Native

---

### 5. 선불 서비스 개발 — 금융당국 라이센스 취득 기여

**개요**
자사 간편결제 선불 서비스를 신규 개발했습니다.

**역할 및 구현**
- 가맹점 API 연동 설계 및 구현
- 금융당국 요건에 맞는 거래 데이터 처리 및 정합성 확보 로직 구현

**성과**
- 금융당국 **선불 라이센스 취득** 기여

---

## 기술 스택 요약

| 구분 | 기술 |
|---|---|
| **Strong** | Java · Spring Boot · JPA · Kafka |
| **Knowledgeable** | Kotlin · MySQL · QueryDSL · Spring Security · Redis · Next.js · React |
| **Etc** | AWS · Docker · Nginx · Jenkins · Vault · Linux · Git |

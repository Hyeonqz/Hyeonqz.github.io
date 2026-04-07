# 경력기술서

> 진현규 · Backend Engineer · jinhyeonkyu@gmail.com
> GitHub: https://github.com/Hyeonqz

---

## (주) 큐뱅 · Server Developer · 2024.01 ~ 현재

핀테크 스타트업. 국내외 간편결제사와 가맹점(POS) 사이의 QR 결제 데이터를 중계·처리하는 B2B 금융 서비스.
서버 개발자로 입사하여 QR 결제 중계 시스템 설계·구현·운영 전 과정을 담당하고 있습니다.

---

### 1. QR 결제 중계 플랫폼 구축 및 운영 · 2024.05 ~ 현재

알리페이·위챗페이·라인페이·유니온페이 등 국내외 간편결제사 **7개사 이상**과 가맹점 POS를 연결하는 결제 중계 시스템을 설계하고 운영했습니다.

**역할**
- REST API Spec **15개 이상** 정의 및 구현 (단독 담당), 결제사별 인증·서명 방식 추상화로 확장 구조 설계
- C++ DLL 기반 레거시 POS 네이티브 모듈 개발 → 이기종 시스템 통합, 월 거래액 **30% 확장**
- 결제 테스트 자동화 웹앱 개발 → 외부기관 커뮤니케이션 비용 **50% 절감**

**기술적 개선**

| 개선 | 배경 | 결과 |
|---|---|---|
| Kafka Request-Reply | 분산 환경 Consumer 불일치로 타임아웃 빈번 발생 | 타임아웃 50%→0%, CS 60%↓ |
| Redis Cache-Aside | Polling 120회 기준 DB 조회 360회 발생 | DB 쿼리 0회, 응답시간 85%↓ |
| Final API 패턴 | POS↔서버 결제 상태 불일치로 CS 월 10건 반복 | CS 95%↓, 연간 40시간 절감 |
| Outbox + SFTP 풀링 | 파일 전송 유실 감지 불가 (유실률 100%) | 유실 0%, 10,000건 무장애 |
| Pinpoint APM | 장애 발생 시 병목 지점 식별 불가 | API 실패율 90%↓ |

**종합 성과:** 일 평균 거래 **50건 → 4,000건 (80배)** 성장 무장애 운영

---

### 2. PCI DSS 컴플라이언스 달성 · 2025.06 ~ 2025.07

- **HashiCorp Vault** 도입 — application.yml 평문 저장 → 중앙집중식 암호화 키 관리, 키 로테이션 자동화
- **Keycloak** 기반 백오피스 2FA 구현, JPA Auditing + Kafka 민감정보 접근 로그 추적 체계 구축
- 보안 취약점 해결: High **5건**, Medium **12건**

**기술스택:** HashiCorp Vault · Spring Cloud Vault · Keycloak · Kafka

---

### 3. 애플리케이션 아키텍처 개선 · 2024.10

- 모놀리식 → Gradle 멀티 모듈 + DDD 기반 헥사고날 아키텍처 전환
- 빌드 시간 **55s → 35s** 단축, Nexus 공통 라이브러리 분리 — 5개 프로젝트 공통 적용

---

### 4. 모바일 POS 시스템 구축 · 2025.10 ~ 2025.12

- Jenkins · Docker · AWS 기반 CI/CD 파이프라인 설계 및 구축
- React Native 기반 크로스 플랫폼 앱 개발 참여

---

### 5. 선불 서비스 개발 · 2024.01 ~ 2024.04

- 자사 간편결제 선불 서비스 가맹점 API 연동, 금융당국 **선불 라이센스 취득** 기여

---

## 기술 스택

| 구분 | 기술 |
|---|---|
| **Strong** | Java · Spring Boot · JPA · Kafka |
| **Knowledgeable** | Kotlin · MySQL · QueryDSL · Spring Security · Redis · Next.js · React |
| **Etc** | AWS · Docker · Nginx · Jenkins · Vault · Linux · Git |




---

## 1. 기술적 도전 경험
### Kafka Request-Reply 아키텍처로 분산 결제 응답 정합성 문제 해결

**문제 상황**

QR 결제 흐름은 POS → 큐뱅 서버(OQ) → 간편결제사(DQ)로 이어지는 구조였습니다.
로컬에서는 정상 동작했지만, 개발 서버를 이중화(2대) 배포한 후 타임아웃이 50% 확률로 발생했습니다.

```text
[Kafka 흐름]
1번 서버 → [requestTopic] → worker 처리
                                  ↓
                            [replyTopic]
                                  ↓
                ┌─────────────────┴─────────────────┐
                ↓                                   ↓
          Gateway 1번 서버                     Gateway 2번 서버
          (요청한 서버, 대기 중)                (엉뚱한 서버가 소비) ❌
                ↓                                   ↓
          응답 못 받음 → 타임아웃!               처리했지만 의미 없음
```

**근본 원인 분석**

두 인스턴스가 **동일한 consumer-group**으로 같은 Reply 토픽을 구독하고 있었던 것이 핵심 원인이었습니다.

Kafka는 동일 consumer-group 내에서 파티션을 인스턴스에 분산 배정합니다.
즉, Reply 토픽의 파티션이 1번 서버와 2번 서버에 나뉘어 할당되므로, 1번 서버가 보낸 요청의 Reply가 2번 서버에 할당된 파티션으로 들어오면 2번 서버가 소비합니다.
1번 서버는 자신의 파티션만 구독하므로 해당 Reply를 영원히 받지 못하고 타임아웃이 발생합니다.

**접근 방법**

결제 요청은 사용자 입장에서 동기적으로 처리되어야 하므로, Kafka를 비동기 메시지 버스로 사용하면서도 **Request-Reply 패턴으로 동기 응답을 보장**하는 구조를 설계했습니다.
Spring Kafka의 `RequestReplyFuture`를 활용해 요청 시 블로킹으로 외부 동기를 유지하고, 내부는 비동기로 동작시켰습니다.

핵심 설계 원칙: *"어느 인스턴스가 Reply를 수신하더라도, 요청을 보낸 인스턴스에 정확히 전달되어야 한다."*

이를 위해 두 가지를 구현했습니다. [(자세한 내용)](https://hyeonq.tistory.com/240)

1. **인스턴스별 전용 Reply 토픽** — 각 인스턴스가 자신만의 Reply 토픽을 구독해 다른 인스턴스의 응답을 절대 수신하지 않도록 격리
2. **KafkaReplyHandler** — 요청 헤더에 담긴 CorrelationId와 Reply 토픽 정보를 추출해 올바른 인스턴스로 응답을 라우팅

아래는 실제 운영 중인 QR 결제 요청 수신 Consumer 코드입니다.

```java
@Slf4j
@Component
public class createPaymentInfoListener implements KafkaConsumerService<ConsumerRecord<String, String>> {

    private final PaymentClientService paymentClientService;
    private final KafkaReplyHandler replyHandler;  // ← CorrelationId 기반 응답 라우팅 담당
    private final ObjectMapper objectMapper;

    @KafkaListener(
        id = "${spring.config.activate.on-profile}-payment-order-verify-request",
        containerFactory = "PaymentKafkaListenerContainerFactory",
        topics = "#{@kafkaTopicProperties.staticQrPaymentTopicList}",  // ← 인스턴스별 전용 토픽
        groupId = "${qrbank.kafka.topics.payment_request.group-id}",
        autoStartup = "true"
    )
    public void subscription(ConsumerRecord<String, String> record, Acknowledgment acknowledgment) {
        // 1단계: 메시지 파싱
        OrderVerifyRequestMessage request = buildOrderVerifyRequestMessage(record);

        // 2단계: Fail-fast 검증 — 잘못된 요청은 즉시 에러 Reply 반환, 거래 처리로 전파 차단
        validConsumerRecordValue(request);

        // 3단계: 실제 거래 정보 조회
        PaymentResponseMessage response = paymentClientService.responseTransactionInfo(request);

        // 4단계: CorrelationId + 전용 Reply 토픽으로 요청 인스턴스에 정확히 라우팅
        replyHandler.sendReply(record, response);

        // 5단계: Reply 전송 완료 후 수동 ACK — 처리 전 ACK로 인한 메시지 유실 방지
        acknowledgment.acknowledge();
    }
}
```

**보완: Cooperative 리밸런싱 전략 도입**

인스턴스별 전용 토픽으로 응답 정합성 문제는 해결했지만, 스케일아웃이나 배포 시 발생하는 **리밸런싱**이 새로운 위험 요소로 남았습니다.

기본 Eager(Stop-the-World) 리밸런싱은 리밸런싱 발생 시 모든 인스턴스가 보유한 파티션을 전부 반납하고, 재할당이 완료될 때까지 전체 소비가 중단됩니다.
결제 시스템에서는 이 중단 시간 동안 인메모리의 `RequestReplyFuture`가 응답을 받지 못해 타임아웃으로 이어질 수 있었습니다.

```text
[Eager 리밸런싱 — 전체 중단]
모든 파티션 반납 → 전체 소비 중단 → 재할당 완료 → 소비 재개
→ 중단 시간 동안 인메모리 Future 타임아웃 위험

[Cooperative 리밸런싱 — 점진적 재할당]
변경 필요한 파티션만 반납 → 나머지는 계속 소비 → 점진적 재할당
→ 리밸런싱 중에도 대부분의 파티션은 정상 처리 유지
```

`CooperativeStickyAssignor`를 적용해 리밸런싱 시 변경이 필요한 파티션만 순차적으로 재할당하도록 했고, 기존에 처리 중인 파티션은 리밸런싱 중에도 소비를 중단하지 않아 인플라이트 결제 요청의 타임아웃 위험을 최소화했습니다.

```yaml
spring:
  kafka:
    consumer:
      properties:
        partition.assignment.strategy: org.apache.kafka.clients.consumer.CooperativeStickyAssignor
```

**결과**
- 타임아웃 발생률 **50% → 0%**
- 리밸런싱 발생 시 인플라이트 결제 요청 타임아웃 위험 최소화

**한계 및 추가 개선 방향**

- 인스턴스 **크래시** 시에는 인메모리 `RequestReplyFuture`가 유실되는 문제가 여전히 남아 있습니다. Cooperative 전략은 정상적인 스케일아웃/배포에는 효과적이지만, 비정상 종료에는 대응하지 못합니다. Redis 기반 분산 상태 저장으로 전환하는 방안을 검토 중입니다.
- 처리량 개선을 위해 Virtual Thread 도입도 병행 검토 중입니다.

---

## 2. 몰입의 순간
### QR 결제 중계 플랫폼 0→1, 그리고 80배 성장까지

2024년 1월 입사 당시 큐뱅의 QR 결제 서비스는 이제 막 시작된 MVP 단계였습니다.
실제 서비스 오픈은 8월이었고, 일 평균 거래 10건에서 출발해 점차 50건으로 늘어나던 시기였습니다.
연동된 결제사는 1개, 모니터링은 수동, 아키텍처 문서는 없었습니다.

저는 알리페이·위챗페이의 API 스펙을 직접 분석하며 연동 레이어를 처음부터 설계했습니다.
각 결제사마다 인증 방식, 에러 코드 체계, 재시도 정책이 모두 달랐고, 이를 하나의 내부 표준 인터페이스로 추상화해 신규 결제사를 손쉽게 추가할 수 있는 확장 구조를 만들었습니다.
이 설계 덕분에 이후 7개사까지 연동을 확장할 때 코드 중복 없이 빠르게 대응할 수 있었습니다.

서비스가 성장하면서 예상치 못한 문제들이 연달아 터졌습니다.
타임아웃으로 거래가 실패하고, 파일이 유실되고, CS 문의가 쌓이고, 확장 불가능한 설계로 개발이 지연됐습니다. <br>
저는 매번 토스·카카오페이 기술 블로그와 오픈소스 아키텍처를 분석해 현재 상황에 맞는 모범 사례를 직접 찾았고, 팀원들과 논의한 뒤 사내 PT 발표로 방향을 확정했습니다. <br>
문제가 생길 때마다 "왜 이게 반복되는가"를 먼저 물었고, 증상이 아닌 구조를 고쳤습니다.

| 개선 | 문제 | 성과 |
|---|---|---|
| Kafka Request-Reply | 분산 환경 타임아웃 반복 | 타임아웃 50%→0% |
| Redis Cache-Aside | DB 과부하, 응답 지연 | 응답시간 85%↓ |
| Final API 패턴 | 결제 상태 불일치 CS 반복 | CS 95%↓, 연간 40시간 절감 |
| Outbox + SFTP 풀링 | 파일 유실 감지 불가 | 유실 0%, 10,000건 무장애 |

결과적으로 서비스는 일 평균 거래 **50건 → 4,000건(80배)**으로 성장했고, 이 과정 전체를 단독으로 설계하고 운영했습니다.

**이 경험에서 배운 것들**

첫째, **구조가 곧 신뢰**라는 것입니다. 결제 시스템에서 하나의 버그를 패치하는 것보다, 그 버그가 발생할 수 없는 구조를 설계하는 것이 팀과 사용자 모두에게 진짜 해결임을 배웠습니다.

둘째, **다기관 연동의 복잡성을 다루는 방법**입니다. 각 결제사의 API 특성과 에러 패턴을 이해하고, 이를 안정적으로 추상화하는 설계 역량을 실전에서 쌓았습니다.

셋째, **기술 결정을 혼자 내리지 않는 것**입니다. 시장 조사 → 팀 공유 → 합의 → 구현의 흐름을 반복하며, 기술 선택에 팀의 신뢰를 얻는 방법을 익혔습니다.

**LINE Pay에서 기여할 수 있는 것**

LINE Pay는 태국 수천만 명의 일상 결제를 처리하는 플랫폼입니다. <br>
저는 국내외 7개 결제사와의 연동 경험, 분산 결제 환경에서의 장애 대응 경험, 그리고 트래픽이 80배 증가하는 과정에서도 안정성을 유지한 설계 경험을 갖고 있습니다. <br>
스타트업에서 혼자 구조를 만들고 부수고 다시 세우며 쌓은 이 밀도는, LINE Pay의 코어 결제 시스템을 설계하고 개선하는 데 즉시 기여할 수 있는 역량이라고 확신합니다.
태국이라는 새로운 시장, 그리고 LINE Pay라는 글로벌 플랫폼 위에서 그 경험을 더 크게 펼치고 싶습니다.

# LINE Pay 포트폴리오

> 진현규 · Backend Engineer · jinhyeonkyu@gmail.com · github.com/Hyeonqz

---

## 1. 기술적 도전 경험
### Kafka Request-Reply 아키텍처로 결제 응답 정합성 문제 해결

**문제 상황**

QR 결제 중계 서비스는 POS → 큐뱅 서버 → 간편결제사(알리페이·위챗페이 등)로 이어지는 분산 결제 흐름에서 Kafka를 메시지 버스로 사용하고 있었습니다.
문제는 멀티 인스턴스 환경에서 Reply 토픽을 수신하는 Consumer가 요청을 보낸 인스턴스와 다를 수 있다는 점이었습니다.
결제 요청을 보낸 인스턴스 A가 응답을 기다리는 동안, Reply는 인스턴스 B가 소비해버려 A는 영원히 응답을 받지 못하고 타임아웃이 발생했습니다.
이로 인해 거래 실패율이 높아지고 수동 CS 대응이 월 5건 이상 반복되었습니다.

**접근 방법**

우선 문제의 핵심을 정리했습니다. "어느 인스턴스가 응답을 받더라도, 올바른 요청자에게 전달되어야 한다."
이를 위해 CorrelationId 기반 Request-Reply 패턴을 설계했습니다.

```java
// 1. 요청 시 CorrelationId 발급 및 Future 등록
@Service
@RequiredArgsConstructor
public class PaymentRequestService {

    private final KafkaTemplate<String, PaymentRequest> kafkaTemplate;
    private final ConcurrentHashMap<String, CompletableFuture<PaymentResponse>>
        pendingRequests = new ConcurrentHashMap<>();

    public PaymentResponse request(PaymentRequest request) throws Exception {
        String correlationId = UUID.randomUUID().toString();
        CompletableFuture<PaymentResponse> future = new CompletableFuture<>();
        pendingRequests.put(correlationId, future);

        kafkaTemplate.send(MessageBuilder
            .withPayload(request)
            .setHeader("correlationId", correlationId)
            .setHeader(KafkaHeaders.TOPIC, "payment.request")
            .build());

        try {
            return future.get(30, TimeUnit.SECONDS); // 타임아웃 30초
        } catch (TimeoutException e) {
            pendingRequests.remove(correlationId);
            throw new PaymentTimeoutException("응답 타임아웃: " + correlationId);
        }
    }

    // 2. 어느 인스턴스가 Reply를 수신해도 CorrelationId로 올바른 Future에 응답
    @KafkaListener(topics = "payment.reply", groupId = "${kafka.consumer.group-id}")
    public void handleReply(ConsumerRecord<String, PaymentResponse> record) {
        String correlationId = new String(
            record.headers().lastHeader("correlationId").value()
        );
        CompletableFuture<PaymentResponse> future = pendingRequests.remove(correlationId);
        if (future != null) {
            future.complete(record.value());
        }
        // future == null 이면 이미 타임아웃 처리된 요청 → 무시
    }
}
```

추가로 타임아웃된 메시지가 누적되지 않도록 DLQ(Dead Letter Queue) 토픽을 별도 설계하고, `@Retryable`로 자동 재시도 흐름을 구성했습니다.

**결과 및 회고**

- 타임아웃 발생률 **50% → 0%**, 수동 CS 월 **5건 → 2건 (60%↓)**
- 한계: `pendingRequests` Map이 인스턴스 메모리에 존재하므로, 인스턴스가 예기치 않게 재시작되면 진행 중인 Future가 유실됩니다. 이를 보완하기 위해 Redis 기반 분산 상태 저장으로 전환하는 개선을 검토 중입니다.

---

## 2. 몰입의 순간
### QR 결제 중계 플랫폼 0→1, 그리고 80배 성장까지

2024년 1월 입사 당시, 큐뱅의 QR 결제 서비스는 MVP 단계였습니다.
일 평균 거래 50건, 1개 결제사 연동, 수동 모니터링이 전부였습니다.

처음에는 연동 스펙 문서 하나 없이 알리페이·위챗페이 API를 직접 분석해 연동 레이어를 설계했습니다. 각 결제사마다 인증 방식, 에러 코드, 재시도 정책이 전부 달랐고, 이를 내부 표준 인터페이스로 추상화해 신규 결제사 추가 시 확장 가능한 구조를 만들었습니다.

하지만 서비스가 성장하면서 예상치 못한 문제들이 터졌습니다.
CS 문의가 쌓이고, 파일 유실이 반복되고, DB가 폭발하고, 타임아웃이 빈번해졌습니다.
저는 이 문제들을 단순히 "처리"하는 것이 아니라, **구조적으로 없애는** 방향으로 접근했습니다.

- Kafka Request-Reply로 타임아웃을 구조적으로 제거
- Redis Cache-Aside로 DB 쿼리 360회 → 0회
- Final API 패턴으로 CS 95% 감소
- Outbox 패턴으로 파일 유실 100% → 0%

각 문제를 해결할 때마다 논문, 결제사 기술 블로그, 오픈소스 코드를 뒤지며 모범 사례를 직접 찾았고, 팀에 사내 PT로 공유하며 합의를 이끌었습니다.

그 결과 서비스는 일 평균 거래 **50건 → 4,000건(80배)** 으로 성장했고, 그 과정 전체를 단독으로 설계하고 운영했습니다.

이 경험에서 가장 깊이 배운 것은 **"장애는 코드의 문제가 아니라 구조의 문제"** 라는 점입니다. 하나의 버그를 고치는 것보다, 그 버그가 발생할 수 없는 구조를 만드는 것이 진짜 해결임을 몸으로 익혔습니다. LINE Pay에서 수천만 명의 결제를 다루는 환경에서 이 사고방식을 더 크게 발휘하고 싶습니다.

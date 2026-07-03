# 진현규 | 서버 개발자

**jinhyeonkyu@gmail.com** · [GitHub](https://github.com/Hyeonqz) · [LinkedIn](https://www.linkedin.com/in/hyeonkyu-jin/) · [Portfolio](https://www.notion.so/3345cb02eeeb80ffafe0d5e80754270a)

---

## About me

핀테크 결제 도메인 2년차 서버 개발자. QR 결제 중계 시스템을 0부터 구축하며 일 평균 거래 **50건 → 4,000건** 성장을 경험했습니다.
운영 이슈를 직접 식별하고 **Kafka 아키텍처 전환, CS 95% 감소** 등 구조적 개선을 주도. 장애 격리·데이터 정합성·트랜잭션 안정성을 최우선으로 설계합니다.

---

## Skill

`Java` `Kotlin` `Spring Boot` `JPA` `Kafka` `MySQL` `Redis`

---

## Work Experience

### (주) 큐뱅 | Server Developer | 2024.01 – 현재

국내·국외 QR 결제 중계 서비스 제공 플랫폼

**[1. QR 중계 시스템 개발 및 운영]** · 국내·국외 QR 결제 중계 서비스

- POS 및 간편결제사 연동 REST API Spec 15개 이상 문서화·구현
- Kafka Request-Reply 아키텍처 설계 — 인스턴스별 고유 Reply Topic 구독으로 분산 환경 라우팅 문제 완전 해소 [(자세한 내용↗)](https://hyeonq.tistory.com/240)
- Redis Cache-Aside + 트랜잭션 경계 최적화 → DB 쿼리 **360회→0회**, 응답시간 **200ms→30ms (85%↓)**
- Final API 패턴 + 멱등 처리 도입 → 결제 상태 불일치 CS 월 **10건→0~1건 (95%↓)**, 연간 **40시간** 수동 공수 절감
- Outbox 패턴 + SFTP 세션 풀링 — DB 저장·PENDING 단일 트랜잭션으로 이벤트 유실 방지, 30분 배치 자동 재처리 → 파일 유실률 **100%→0%** [(관련 내용↗)](https://hyeonq.tistory.com/243)
- C++ DLL 기반 레거시 POS 네이티브 모듈 직접 개발 → 이기종 시스템 통합, **월 거래액 30% 확장**

**[2. 기타 서비스 개발]**

- 모바일 POS 서버 아키텍처 설계 및 ReactNative 기반 크로스 플랫폼 앱 개발
- 자사 간편결제 선불 서비스 가맹점 API 연동 · 금융당국 선불 라이센스 취득 기여
- 전자 계약서 생성 유료 솔루션 → 오픈소스 전환, **연 $999 비용 절감**

---

## Open Sources

**es-hangul-java** &nbsp; 토스의 한글 처리 라이브러리(es-hangul)를 Java로 포팅 후 Maven Central 배포 및 사내 시스템 적용

---

## Activities

| | | |
|---|---|---|
| **SLiPP** | 2026.03 – ing | 백엔드 개발자 커뮤니티 활동 · Claude Code 기반 AI 개발 워크플로우 스터디 진행 |
| **오픈소스 개발자 대회** | 2025.07 – 08 | seoul-fit : 서울시 공공데이터 API 10개 이상 연동 및 ETL 파이프라인 담당 |
| **쌍용교육센터** | 2023.06 – 12 | AWS·Docker 활용 Java FullStack 과정 수료 · 팀 프로젝트 2회 협업 경험 |

---

## Certificate · Education

| | |
|---|---|
| **정보처리기사** &nbsp; 한국산업인력공단 2024.06 | **남서울대학교** &nbsp; 드론공간정보공학 (학사) 2018.03 – 2024.02 |

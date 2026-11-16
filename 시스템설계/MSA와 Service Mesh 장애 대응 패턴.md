# MSA와 Service Mesh 장애 대응 패턴

> 서비스가 많아질수록 호출/장애 전파/설정 변경을 어떻게 안전하게 관리할지 정리

## 1. 서비스 간 호출과 추적

- **호출 방식**: WebClient(Netty 기반 Non-blocking)는 적은 스레드로 높은 동시성을 처리하고, RestClient/RestTemplate은 서블릿 스레드를 블로킹하는 동기 호출이다 — 호출량과 하위 자원(스레드풀 크기) 제약에 따라 선택한다. 서비스 간 호출을 표준화하기 위해 SDK를 자체 제작할지(버전 관리·인증·재시도를 캡슐화) 여부도 의사결정 포인트
- **분산 트레이싱**: client가 trace id/span id를 생성해 요청 헤더로 릴레이(W3C `traceparent`, Zipkin `B3`). 모든 서비스가 받은 헤더를 그대로 다음 호출에 전달해야 Zipkin/Pinpoint 같은 도구에서 전체 호출 체인을 하나로 이어 볼 수 있다

## 2. 연쇄 장애(cascading) 차단

장애 전파를 막는 4가지 패턴은 함께 조합해서 쓴다.

| 패턴 | 하는 일 |
|------|---------|
| **circuit breaker** | 특정 호출의 실패율이 임계치를 넘으면 회로를 열어 재시도 폭주를 막음 |
| **global circuit breaker** | 개별 API가 아니라 의존 서비스 전체 레벨에서 장애를 감지 |
| **bulkhead** | 호출 대상별로 스레드풀/커넥션풀을 격리해 한쪽 장애가 다른 쪽에 안 번지게 함 |
| **rate limit** | 호출 측에서 처리 가능한 양 이상 요청을 보내지 않도록 제한 |

## 3. 서비스 합치기/분리하기 전 검증

실제 트래픽에 영향을 주지 않으면서 결과를 미리 확인하는 방법들.

- **shadow traffic**: 트래픽을 복제해 신규 로직에도 흘려보내되 결과는 버림
- **CDC(Change Data Capture)**: DB 변경 이력을 캡처해 다른 시스템에 동기화 — 데이터 이관 검증에 사용
- **dry-run**: 실제 쓰기 없이 로직만 수행해 결과를 미리 확인
- **dataset 효율 검증**: 전수 비교 대신 aggregate-count, sum, hash 같은 요약값으로 두 시스템의 데이터 일치 여부를 빠르게 비교

## 4. Resilience 로직의 위치: application vs mesh

circuit breaker, retry, rate limit을 **애플리케이션 코드**에 둘지 **서비스 메시(Istio/Envoy 사이드카)**에 둘지는 세밀한 제어와 운영 일관성의 트레이드오프다.

| 기준 | Application 레벨 | Mesh 레벨 |
|------|-------------------|-----------|
| 세밀한 제어 | 비즈니스 로직별 커스터마이징 용이 | 서비스 단위로 획일적 |
| 운영 부담 | 코드/라이브러리 버전 관리 필요 | 인프라 팀이 중앙에서 일괄 관리 |
| 언어 독립성 | 언어별로 구현해야 함 | 언어 무관하게 사이드카가 처리 |
| 가시성 | 코드를 봐야 알 수 있음 | 메시 설정만 보면 전체 정책 파악 가능 |

## 5. 설정의 실시간 전파

배포 없이 설정값을 바꾸는 **dynamic config** 구조가 필요하다. 설정 서버는 목적에 따라 갈린다.

- **Central Dogma**: Git 기반 버전 관리 + 미러링
- **Consul KV**: Service Discovery와 함께 Key-Value 설정 저장소 제공
- **Spring Cloud Config + Bus**: Git을 설정 소스로 쓰고, 변경 시 메시지 버스(Kafka/RabbitMQ)로 전체 인스턴스에 refresh 전파

## 6. 장애 복구를 넘어 예방으로: fault injection

의도적으로 지연/에러를 주입하는 Chaos Engineering으로, circuit breaker·retry·failover가 실제 장애 상황에서 제대로 동작하는지 사전 검증한다. 목표는 장애를 "복구"하는 것을 넘어 **미리 예방**할 수 있는지까지 확인하는 것이다.

## 핵심 정리

- 서비스 간 통신은 호출 방식 선택부터 trace 전파까지 일관된 표준이 있어야 장애 추적이 가능하다
- 연쇄 장애를 막는 4대 패턴(circuit breaker, global circuit breaker, bulkhead, rate limit)은 조합해서 써야 효과가 있다
- resilience 로직을 app과 mesh 중 어디에 둘지는 정답이 없고, 세밀한 제어 vs 운영 일관성의 트레이드오프다
- Chaos Engineering(fault injection)으로 장애 대응 체계를 "사고 나기 전에" 검증하는 문화가 중요하다

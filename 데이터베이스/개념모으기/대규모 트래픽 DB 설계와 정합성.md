# 대규모 트래픽 DB 설계와 정합성

> write가 늘고 데이터가 커질 때 DB를 어떻게 확장하고, 정합성을 어떻게 지킬지 정리

## 1. 쓰기/용량 확장: sharding과 partition

- **sharding**: 하나의 DB가 감당 못 할 쓰기량을 여러 인스턴스로 수평 분할(유저 ID 등으로 라우팅)
- **MySQL → Cassandra**: 쓰기 위주·대량 로그성 데이터는 RDBMS보다 쓰기 스케일아웃이 쉬운 Wide-Column 스토어로 이관
- **StarRocks**: 대용량 분석/집계 쿼리는 OLAP 특화 엔진으로 분리해 운영 DB의 분석 부하를 덜어냄
- **partition**: 테이블을 range/hash/list 기준으로 물리적으로 나눠, 쿼리 시 필요한 파티션만 스캔(pruning)하고 보관 주기 삭제 같은 관리도 쉬워짐

## 2. 조회 성능: index와 쿼리 가시성

- **Index / composition index**: WHERE·JOIN·ORDER BY에 쓰이는 컬럼에 인덱스를 태워 풀스캔을 방지. 복합 인덱스는 **컬럼 순서(leftmost prefix)**가 중요 — 선행 컬럼이 조건절에서 먼저 매칭돼야 인덱스를 탄다
- **hibernate intercept**: Interceptor/StatementInspector로 실행 쿼리에 **호출한 API path를 comment로 주입** — 느린 쿼리 로그만 봐도 어떤 API에서 왔는지 바로 추적되어 장애 파악이 빨라진다
- **select query type 분류**: 단건/리스트/집계 조회 패턴을 분류해 어떤 유형이 부하를 많이 일으키는지 통계화

## 3. 장애 대응: 이중화

Master-Replica 이중화로 장애 시 Replica를 승격(failover)하는 것만으로는 부족하다. **복제 지연으로 인한 데이터 유실 가능성**과 **애플리케이션의 재연결 처리**까지 포함해 "failover 시 실제로 어떻게 복구되는지"를 이해하고 있어야 실제 장애 때 당황하지 않는다.

## 4. 정합성: 멱등성과 락

- **idempotency key 구현 위치**
  - DB: unique constraint로 중복 insert 차단 — 재시작/재배포에도 안전하지만 DB 부하 발생
  - memory: 캐시에 처리 여부 기록 — 빠르지만 인스턴스 재시작 시 유실 위험
- **lock 동작 방식**: spin(짧은 간격 재시도, 락 보유 시간이 짧을 때 유리) vs waiting(대기 큐, CPU 소모 적음)
- **lock 구현 레벨**: Optimistic(버전 컬럼으로 커밋 시점 충돌 감지, 충돌이 드물 때 유리) vs Pessimistic(조회 시점에 락을 걸어 충돌 자체 차단, 충돌이 잦을 때 유리)
- **이기종 transaction (DB + Kafka)**: DB 커밋과 Kafka 발행은 하나의 트랜잭션으로 묶이지 않으므로, **Outbox 패턴**(DB에 이벤트를 같이 커밋하고 별도 프로세스가 Kafka로 발행)으로 원자성을 흉내낸다

## 5. 분산락: 서비스 경계를 넘는 동시성 제어

같은 자원(재고 차감, 포인트 적립 등)에 여러 서비스 인스턴스가 동시에 접근할 때는 프로세스 내 락으로 막을 수 없어 외부 저장소에 위임한 분산락이 필요하다.

- **DB·Redis 기반 Set NX**: `SET key value NX PX ttl`로 "키가 없을 때만 생성"하는 원자적 연산을 락으로 활용
- **RedLock**: 독립된 여러 Redis 노드 과반수(quorum)에서 락 획득에 성공해야 유효

실제 구현 시 반드시 고려할 것: **인스턴스 장애**(락을 잡은 노드가 죽으면 TTL 전까지 락이 안 풀릴 수 있음), **major 획득 실패**(과반수 실패 시 재시도 전략), **NTP**(노드 간 시계가 어긋나면 TTL 계산이 틀어짐), **TTL 만료**(작업이 TTL보다 길어지면 락이 풀린 뒤에도 작업이 이어짐), **락 획득 시간**(잡는 데 걸린 시간을 빼지 않으면 TTL이 거의 다 된 락을 유효하다고 착각할 수 있음).

> 더 깊은 메커니즘(ZooKeeper 기반 분산락, Fencing Token, RedLock 비판)은 [[분산락 구현 원리 (Zookeeper, Redlock)]] 참고

## 핵심 정리

- 쓰기 확장은 sharding/partition, 조회 확장은 index·전용 분석엔진 분리로 해결한다
- API 단위로 쿼리를 추적할 수 있게 만들어두면(hibernate intercept) 장애 원인 파악 속도가 크게 빨라진다
- 정합성은 "멱등성을 어디서 보장할지(DB vs memory)"와 "어떤 락 전략을 쓸지(spin/waiting, 낙관/비관)"의 조합으로 설계한다
- 분산락은 SET NX로 끝나는 게 아니라 **시계 동기화·TTL·장애 시나리오**까지 고려해야 안전하다

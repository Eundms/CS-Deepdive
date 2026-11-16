# Kafka 비동기 처리 아키텍처와 운영

> 대용량 트래픽에서 Kafka 컨슈머/브로커를 어떻게 설계하고 운영할지 정리

## 1. 컨슈머 처리량 고도화

```
record 단위 → batch 단위 → multi-thread → parallel consumer
```

처리량을 높이는 과정은 보통 이 네 단계를 거친다. record 단위(하나씩 처리)는 가장 단순하지만 느리고, batch 단위로 모아 처리하면 I/O 횟수가 줄며, 파티션별 multi-thread로 병렬화하면 처리량이 코어 수만큼 늘어난다. parallel consumer는 한 파티션 **내부**에서도 key 단위로 더 쪼개 병렬 처리한다.

단계가 올라갈수록 책임도 커진다.
- **순서 보장**: 같은 key(동일 유저 등)의 메시지는 순서가 지켜져야 하므로 병렬화 단위를 key hash로 맞춰야 함
- **rebalance 대응**: 처리 중 파티션이 재할당되면 in-flight 메시지를 안전하게 마무리하고 커밋해야 함
- **부분 실패 처리**: 병렬 처리 중 일부만 실패하면 offset 커밋이 더 이상 단순 선형이 아니게 됨

## 2. 실패 처리: DLQ와 재발행

일정 횟수 재시도해도 실패한 메시지는 **DLQ(Dead Letter Queue)**로 격리해 메인 파이프라인이 막히지 않게 한다. DLQ는 쌓아두는 것으로 끝이 아니라, 원인 분석 후 원래 토픽으로 **재발행(replay)**할 수 있는 운영 도구가 같이 있어야 실질적인 복구 수단이 된다.

## 3. 브로커 성능: 배치와 압축

- **배치(batch)**: producer의 `batch.size`, `linger.ms`로 네트워크 요청 횟수를 줄임
- **압축(compression)**: `compression.type`(gzip/snappy/lz4/zstd)으로 네트워크·디스크 I/O를 줄이되 CPU를 더 씀

두 설정을 함께 조정해 **처리량과 지연시간의 균형점**을 찾는 것이 버퍼 최적화의 본질이다.

## 4. 파티션 증설과 오프셋 정책

파티션을 늘리면 새 파티션에는 기존 컨슈머 그룹의 offset이 없어 `auto.offset.reset` 정책을 타게 된다. `latest`로 두면 증설 직후 유입된 데이터를 놓칠 수 있으므로, 누락 없이 처리를 시작하려면 상황에 따라 `earliest`로 전환해야 한다.

## 5. 멱등성 보장: 중복 처리 방지

Kafka는 "최소 한 번(at-least-once)" 전달을 기본으로 하므로 재처리로 인한 중복은 애플리케이션이 직접 막아야 한다.

- **idempotency key**: 메시지에 고유 키를 부여하고 처리 전 DB/캐시에서 처리 여부를 확인(또는 unique constraint)
- **exactly-once 발송**: 알림 발송처럼 외부 부수효과가 남는 작업은 멱등 키 없이는 중복 발송 위험이 큼
- **materialized view**: 집계·조회 결과를 미리 계산해 저장하면 재처리가 일어나도 최종 상태가 덮어쓰기(idempotent)되도록 설계 가능

## 6. 무중단 배포: assigner와 graceful shutdown

- **assigner**: 리밸런싱 시 파티션 재분배 전략(Range, RoundRobin, Sticky, CooperativeSticky) — Sticky 계열이 리밸런싱 비용을 줄임
- **graceful shutdown**: Kubernetes는 먼저 SIGTERM을 보내고 유예시간(`terminationGracePeriodSeconds`) 후 SIGKILL을 보낸다. 그 안에 처리 중인 메시지를 끝내고 커밋해야 배포 중 데이터 유실/중복을 막을 수 있다

## 7. 모니터링과 장애 대응

컨슈머 그룹의 **lag**(생산 offset과 소비 offset의 차이)이 처리 지연을 가장 직접적으로 보여준다. producer/consumer 핵심 설정(`acks`, `retries`, `max.poll.interval.ms` 등)을 장애 시나리오별로 점검하고, lag이 쌓였을 때의 후속 대책(스케일 아웃, 파티션 증설, 처리 로직 최적화)을 미리 플레이북으로 정리해둬야 대응이 빠르다.

## 핵심 정리

- 컨슈머 처리량은 record → batch → multi-thread → parallel 순으로 고도화하되, 병렬화할수록 순서 보장·리밸런싱 책임이 커진다
- 멱등성은 Kafka가 보장해주지 않으므로 애플리케이션이 idempotency key로 직접 구현해야 한다
- 배포/장애 대응의 핵심은 결국 **lag 모니터링과 graceful shutdown**을 제대로 관리하는 것이다

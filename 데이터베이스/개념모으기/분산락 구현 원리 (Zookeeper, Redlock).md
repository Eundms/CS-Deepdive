# 분산락 구현 원리
> 공통 서비스로 분산락 제공할 때 필요

## 분산 락이 필요한 이유 (동시성 문제)
## ZooKeeper 기반 분산 락 (Ephemeral + Sequential Node)
## Redis 기반 분산 락 (SETNX, Redlock 알고리즘)
## Redlock의 한계와 Martin Kleppmann의 비판
## 분산 락 vs 낙관적 락 vs DB 락 비교
## Fencing Token 패턴

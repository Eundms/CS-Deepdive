# Failover(장애 조치)
> 서버나 시스템에서 장애가 발생했을 떄 자동으로 대체 시스템으로 전환하여 서비스 중단 없이 계쏙 운영되도록 하는 메커니즘

- Passive Failover: 대기 서버가 대기만 하다가 장애 시 전환
- Active-Active: 여러 서버가 동시에 작동하며, 하나가 죽으면 다른 노드가 그 역할을 수행
- Hot/Warm/Cold Standby: 대기 서버의 준비 상태에 따라 구분

# Replication (복제)
> 데이터를 여러 서버/노드에 복제하여 가용성과 성능, 데이터 안전성을 높이는 기술

- Master-Slave (Primary-Replica)
    - 쓰기는 Master에서만 읽기는 Slave에서
    - MySQL, PostgreSQL 등에서 자주 사용
- Multi-Master
    - 여러 노드가 읽기/쓰기를 모두 가능
    - 충돌 해결(Conflict Resolution)이 필요
- Peer-to-Peer
    - 모든 노드가 동등한 권한을 가짐

# Consistency 모델
> 분산 시스템에서 데이터 일관성을 어떻게 보장할 것인가에 대한 모델

- Strong Consistency : 모든 노드에서 항상 동일한 값 - RDBMS
- Eventual Consistency : 시간 지나면 모든 노드가 결국 동일한 값 - Cassandra
- Casual Consistency : 인과관계 있는 업데이트는 순서 보장 - 일부 NoSQL
- Read-your-writes : 자신이 쓴 데이터는 바로 읽을 수 있음 - 사용자 경험 강화 


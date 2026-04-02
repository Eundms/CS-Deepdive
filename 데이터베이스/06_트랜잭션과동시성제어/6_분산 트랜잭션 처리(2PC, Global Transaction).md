# 분산 트랜잭션 처리 
여러 개의 서로 다른 데이터베이스 또는 시스템에서 실행되는 트랜잭션을 하나의 트랜잭션처럼 관리하는 방식

## 2PC(Two-Phase Commit)
분산 트랜잭션을 처리할 때 모든 노드가 트랜잭션을 수행할 준비가 되었는지 확인한 후 최종적으로 커밋하거나 롤백하는 방식  
`Coordinator(조정자)`와 `Participant(참여자)` 간의 통신을 통해 트랜잭션을 조정  

1. Prepare Phase (준비 단계): Coordinator가 모든 Participant에게 트랜잭션 준비 요청  
- 각 Participant는 트랜잭션 수행 후 로그를 남기고 `준비 완료(Ready)` 또는 `실패(Failed)` 메시지를 반환  
2. Commit Phase (커밋 단계): 모든 Participant가 준비 완료 상태면 트랜잭션을 커밋  
- 하나라도 실패하면 전체 트랜잭션을 롤백  

> !이종간 DB에서의 분산 트랜잭션 지원 및 구현은 까다로움 

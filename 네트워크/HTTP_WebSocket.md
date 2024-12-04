## HTTP (HyperText Transfer Protocol) vs WebSocket
- HTTP 
    - 단방향 통신
    - 비연속적 연결
    - 상태 비저장(Stateless)방식 : 각 요청을 독립적으로 처리하고 이전 요청과의 상태를 저장하지 않음
    - HTTPS
- WebSocket
    - 양방향 통신
    - 지속적 연결
    - 상태 저장 방식 : 연결이 지속되므로 클라이언트와 서버가 서로의 상태를 추적하고 실시간으로 상태를 갱신할 수 있음
    - WSS

- STOMP
    - 메시징 프로토콜 : WebSocket을 통해 메시지를 주고받을 때 사용되는 프로토콜, 메시지 큐와 Pub/Sub 모델을 지원
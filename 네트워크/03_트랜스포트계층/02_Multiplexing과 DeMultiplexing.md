
# Header + Data(Payload)
- Application Layer : 애플리케이션 목적 기능 제공
- Transport Layer : 프로세스 - 프로세스
    - TCP segment
    - UDP datagram
- Internet Layer : 호스트 - 호스트 
    - IP Datagram, IP Packet
- link layer 
    - Frame
    - Header payload tralier


# DeMultiplexing vs Multiplexing
> 트랜스포트 계층(4)은 네트워크 계층(3)으로부터 세그먼트를 수신하여 호스트에서 동작하는 해당 애플리케이션 프로세스에게 세그먼트의 데이터를 전달하는 의무를 가짐

- DeMultiplexing
Internet layer로부터 받은 segment나 datagram에 있는 payload를 적절한 socket으로 전달하는 것

- Multiplexing
여러 소켓들로부터 데이터를 수집해서 각각 segment나 datagram으로 만든 후 Internet layer로 내려보내는 것
(캡슐화)

 
## `비연결형 다중화/역다중화`와 `연결형 다중화/역다중화` 서비스 
- 소켓은 유일한 식별자를 가짐
- 비연결형 다중화/역다중화
    - 소스 포트 번호, 목적지 포트 번호
    - 오버헤드 적음
    - 스트리밍, DNS, VoIP
- 연결형 다중화/역다중화
    - 소스 IP, 목적지 IP, 소스 포트, 목적지 포트
    - 오버헤드 높음
    - 파일 전송, HTTP 요청, 이메일
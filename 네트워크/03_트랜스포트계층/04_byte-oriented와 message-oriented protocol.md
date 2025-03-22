# message-oriented protocol
- `보낸 메시지 단위 그대로` 받을 수 있게 해주는 프로토콜
- 메시지 간의 경계를 구분할 수 있음
- 한번에 전송 가능한 메시지의 최대 크기가 존재함 
    - 더 큰 메시지를 전송하려고 하면 실패하고 에러를 받는다 

## UDP -> message-oriented protcol
sendto("This")
sendto("is")
sendto("UDP")

This recvfrom()
is recvfrom()
UDP recvfrom()


# byte-oriented protocol (stream-oriented protocol)
- byte들의 흐름으로 인식하고 전송함 
- 메시지의 경계나 구분에 관심이 없음 


## TCP -> byte-oriented protocol
- TCP Segment 단위로 보냄
    - TCP는 신뢰할 수 있는 데이터 전송(reliable)을 실현하는 것이 목적
    - 애플리케이션에서 보낸 메시지와는 독립적으로 특정 크기(MSS; Maximum Segment size)로 구분되는 별도의 전송 단위를 통해 데이터를 전송하게 됨
    - 만약 MSS를 넘어가면 MSS에 맞게 TCP가 잘라서 보내게 되고 수신자 측에서 의미 단위로 재조합해야 함 

## Q. byte-stream protocol을 사용하는 쪽에서 메시지를 구분하는 방법
1. 메시지 사이즈 고정 
2. 메시지 사이즈를 먼저 보낸 후 메시지를 보내기
3. 메시지 사이에 구분자를 넣어서 전송 (CRLF)

## Q. TCP 위에서 동작하는 HTTP는 자체적으로 HTTP Message의 경계를 구분하는 방법을 가지고 있는가 
- YES; HTTP 버전에 따라 달라짐
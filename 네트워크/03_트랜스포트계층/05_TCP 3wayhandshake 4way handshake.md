# 4way handshake
[클라이언트]                                 [서버]
     ESTABLISHED                            ESTABLISHED
         |                                       |
   ① FIN ---------------------------------->    |  ← 연결 종료 요청
         |                                       |
         |                            ② ACK <---  ← 요청 수락 (half-close)
         |                                       |
         |                            ③ FIN --->  ← 서버도 종료 요청
         |                                       |
   ④ ACK <---------------------------------      ← 수락
         |                                       |
         |                                       |
     TIME_WAIT                                  CLOSED
   (2 * MSL 동안 대기)                      
         |
         ↓
      CLOSED


① 클라이언트가 FIN 전송	"이제 나 보낼 거 없어요"
② 서버가 ACK 전송	"알겠어요, 그럼 나머지 보내고 닫을게요"
③ 서버가 FIN 전송	"나도 이제 다 보냈어요"
④ 클라이언트가 ACK 전송	"네, 확인했어요" → 이 시점부터 TIME_WAIT 진입



## RST 패킷의 특징

[Client] ------ RST --------> [Server]


3-way / 4-way handshake 생략 → 그냥 바로 연결 강제 종료

수신 측은 보통 "Connection reset by peer" 같은 메시지로 확인 가능

TCP 상태 변화: 연결 관련 상태 없이 바로 CLOSED로 바뀜

TIME_WAIT 없음 → 빠르게 자원 회수 가능


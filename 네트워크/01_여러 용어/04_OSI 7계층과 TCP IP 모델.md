# OSI 7계층 
> encapsulation, decapsulation 

## Physical (물리 계층)
    - 데이터 전송을 위한 하드웨어적 연결과 전송 (케이블, 신호)
    - bits 단위로 데이터 전송 

## Data Link (데이터 링크 계층) : 프레임(frame)
    - 직접 연결된 노드 간의 통신 담당
    - MAC 주소 기반 통신 (ARP)
- 송신 노드로부터 하나의 링크를 통해 반대편에 있는 수신 노드까지 신뢰적 전송을 제공


-------------------- (Network Access Layer)


## Network (네트워크 계층) : 데이터그램(datagram)
    - 호스트 간의 통신 담당 (IP)
    - 네트워크 간의 최적의 경로 결정 


-------------------- (Network Layer)


## Transport (전송 계층) : 세그먼트(segment)
    - 애플리케이션 간의 통신 담당
    - 목적지 애플리케이션으로 데이터 전송
    - 안정적이고 신뢰할 수 있는 데이터 전송 보장 (TCP)
    - 필수 기능만 제공 (UDP)


-------------------- (Transport Layer)


## Session (세션 계층)
    - 통신 세션 설정, 관리, 종료
        - RPC 

## Presentation (표현 계층)
    - 애플리케이션 간의 통신에서 메시지 포멧 관리 
        - 인코딩 <> 디코딩
        - 암호화 <> 복호화
        - 압축 <> 압축풀기

## Application (응용 계층)
    - 애플리케이션 목적에 맞는 통신 방법 제공
    - HTTP, DNS, SMTP, FTP 


-------------------- (Application Layer)



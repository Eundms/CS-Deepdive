# TCP Fast Open (TFO) : 연결 중 데이터 전송 

TCP의 3-way handshake를 단축해서, 연결과 동시에 데이터 전송 가능하게 만든 기술

## 기존 TCP
[SYN] → [SYN-ACK] → [ACK + DATA]

## TFO 
[SYN + DATA] → [SYN-ACK + DATA] → [ACK]


# HTTP Keep-Alive : 하나의 TCP로 여러 요청

TCP 연결을 재사용하여 여러 HTTP 요청을 처리하는 기술 
TCP 연결은 3-way handshake → 시간+리소스 소비

매번 연결 끊고 새로 연결하면 느림

HTTP/1.0: Connection: keep-alive 헤더로 요청
HTTP/1.1: 기본적으로 Keep-Alive가 기본 설정
하나의 연결로 이미지, JS, CSS 등 다수의 리소스 요청 처리  


# TLS 0-RTT : 암호화된 데이터 즉시 전송
이전에 통신했던 서버와는 "0-RTT"로 암호화된 데이터를 즉시 보낼 수 있음

## 일반 TLS 핸드셰이크
TLS 1.2: 최소 1.5 RTT 소요 (연결 + 키교환)

TLS 1.3: 기본 1 RTT

0-RTT: 첫 메시지부터 데이터 동시 전송 가능
→ 마치 TCP Fast Open처럼 빠르게 처리됨

## 동작 원리
클라이언트가 서버와 과거에 통신한 적이 있음 → "세션 키 재사용"

ClientHello + EarlyData 동시에 전송

## 단점
재전송 공격 가능성 (replay attack)
→ 중요한 데이터(결제, 인증 등)는 0-RTT로 보내지 않음
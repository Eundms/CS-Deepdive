# HTTP 연결과 네트워크 장애 트러블슈팅

> 서버 간 HTTP 통신에서 생기는 연결 문제와 간헐적 장애를 어떤 도구로 추적할지 정리

## 1. HTTP 연결 설정값 불일치

연결 문제의 상당수는 구간별 타임아웃 값이 서로 맞지 않아서 생긴다.

- **connection timeout**: TCP 커넥션을 맺는 데 걸리는 시간 제한 — 이 안에 3-way handshake가 끝나지 않으면 실패
- **idle timeout**: 커넥션을 맺은 뒤 요청이 없을 때 서버/로드밸런서가 끊는 제한. 클라이언트 idle timeout이 서버보다 길면, 서버가 이미 끊은 죽은 커넥션으로 요청을 보내 에러가 난다
- **keep-alive**: 매 요청마다 핸드셰이크를 다시 하지 않도록 커넥션을 재사용 — keep-alive 값과 idle timeout 값이 어긋나면 "connection reset" 류의 간헐적 오류가 생긴다
- **content encoding**: `Accept-Encoding`/`Content-Encoding` 헤더가 어긋나면 압축(gzip 등)된 응답을 못 풀어 파싱 실패

## 2. 간헐적 끊김: 커널·네트워크 레이어 추적

애플리케이션 로그만으로는 원인이 안 보일 때, 아래 순서로 레이어를 내려가며 확인한다.

1. **tcpdump**로 실제 패킷을 캡처해 어느 구간에서 끊기는지(RST, FIN, 재전송) 확인
2. **conntrack**(커널 connection tracking 테이블)이 한도를 넘으면 새 연결이 거부되거나 기존 연결이 끊김 (`conntrack -L`로 현황 확인)
3. **iptables** NAT/필터링 규칙이 의도치 않게 패킷을 drop하고 있는지 확인
4. **ipvs**(kube-proxy의 L4 로드밸런싱 모드) 규칙/타임아웃 설정이 원인인지 확인

## 3. IP 주소 조회 실패: 캐시 계층 확인

DNS 관련 이슈는 "분명히 바꿨는데 일부만 옛 IP로 간다"는 증상으로 나타나는데, 캐시가 여러 겹이기 때문이다.

```
JVM cache → local DNS(nscd 등) → DNS 서버
```

- **JVM cache**: `InetAddress`는 기본적으로 DNS 조회 결과를 캐싱(`networkaddress.cache.ttl`) — 무한(-1)이면 JVM 재시작 전까지 옛 IP를 계속 사용
- **local DNS / DNS 서버**: 각 레이어에 캐시가 있어 어디가 stale한지 단계적으로 확인 필요
- **TTL**: DNS 레코드 TTL이 길면 IP 변경(장애 조치, 인프라 교체) 전파가 느려짐

## 4. 트래픽 라우팅: proxy와 service mesh

- **proxy**: 클라이언트와 서버 사이에서 요청을 중계해 인증·로깅·라우팅을 한 곳에서 처리
- **service mesh**: 사이드카(Envoy 등)가 서비스 간 트래픽을 가로채 라우팅/재시도/mTLS를 담당
  - **shadow request**: 실제 트래픽을 복제해 신규 버전/다른 서비스로도 흘려보내 실제 응답에는 영향 없이 검증
  - **routes**: 헤더·가중치 기반으로 트래픽을 여러 목적지로 분기(카나리 배포, A/B 테스트)

## 핵심 정리

- 연결 문제의 대부분은 **타임아웃 값의 불일치**(클라이언트 vs 서버 vs 중간 장비)에서 온다 — 구간별 timeout/idle 값을 표로 정리해두면 디버깅이 빨라진다
- 간헐적 장애는 애플리케이션보다 **커널/네트워크 레이어(conntrack, iptables, ipvs)**에서 원인을 찾아야 하는 경우가 많다
- DNS 이슈는 JVM 캐시 → OS local DNS → 실제 DNS 서버까지, 캐시가 여러 겹이라는 점을 항상 염두에 둔다

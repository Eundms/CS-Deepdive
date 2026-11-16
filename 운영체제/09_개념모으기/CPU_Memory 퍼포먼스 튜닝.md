# CPU/Memory 퍼포먼스 튜닝

> 운영 중인 서버에서 CPU/메모리 병목을 어떤 도구로 찾고, 어떻게 줄일지 정리

## 1. CPU 병목 추적 도구

- **async profiler**: JVM에 attach해 flame graph를 생성 — 어떤 메서드 호출 스택이 CPU를 많이 점유하는지 시각화
- **perf**: 리눅스 커널 레벨 프로파일러로, 애플리케이션뿐 아니라 커널 함수까지 포함한 전체 CPU 사용을 분석
- **eBPF**: 커널에 안전하게 커스텀 프로브를 삽입해 실시간 커널 이벤트를 추적
  - `runqslower`: 스레드가 CPU 런큐에서 얼마나 오래 기다리는지(스케줄링 지연) 측정
  - `perf sched`: 컨텍스트 스위치, 마이그레이션 같은 스케줄러 이벤트 추적

애플리케이션 프로파일(async profiler)과 커널 레벨(perf, eBPF)을 함께 봐야 원인을 놓치지 않는다.

## 2. CPU 병목의 실제 사례

- **context switching 과다**: 스레드 수가 코어 수 대비 과도하면 체감 CPU 사용률은 높은데 실제 처리량은 떨어짐
- **jemalloc pairing heap 문제**: 메모리 allocator 내부 자료구조 연산이 특정 패턴에서 CPU를 과도하게 소모
- **hypervisor CPU steal**: 클라우드 VM에서 물리 코어를 다른 VM과 나눠 쓰면, 내 프로세스는 멀쩡해도 hypervisor가 CPU를 뺏어가(steal time) 처리 속도가 떨어짐 — `vmstat`의 `st` 컬럼으로 확인

## 3. 메모리 병목 추적

메모리 문제는 JVM 힙만 보면 놓치기 쉽다. 아래 네 지점을 함께 본다.

- **heap dump**: 어떤 객체가 메모리를 점유하는지(누수, 거대 캐시) 분석
- **GC**: GC 로그/튜닝으로 Old Gen 적체나 Full GC 빈도를 확인
- **container 요청 메모리(RSS, inode)**: 컨테이너의 cgroup은 RSS 기준으로 메모리를 보는데, JVM 힙 설정과 컨테이너 limit을 맞추지 않으면 OOMKilled 발생. inode(파일 개수) 고갈도 디스크 용량은 남았는데 쓰기 실패를 일으킬 수 있음
- **mmap과 page cache**: mmap은 페이지 캐시를 많이 소모할 수 있어, `echo 3 > /proc/sys/vm/drop_caches`로 캐시를 비워 애플리케이션이 실제 쓰는 메모리와 구분해서 파악해야 함

## 4. 전송/저장 비용 절약

- **압축**: gzip, zstd로 네트워크/디스크 전송량을 줄이되 CPU 비용과의 트레이드오프를 고려해 일정 크기 이상만 압축하는 **threshold**를 둠
- **직렬화 포맷**: avro, protobuf 같은 바이너리 포맷은 JSON보다 훨씬 작아 전송량·파싱 비용이 줄어듦
- **스키마 운영**: 바이너리 포맷은 스키마가 바뀌면 호환성 문제가 생기므로, 스키마 레지스트리로 상위/하위 호환성을 관리해야 함

## 핵심 정리

- CPU 병목은 애플리케이션 프로파일과 커널 레벨을 함께 봐야 하며, 클라우드 환경에서는 hypervisor steal time처럼 애플리케이션 밖의 원인도 있다
- 메모리 문제는 JVM 힙뿐 아니라 **컨테이너 메모리 limit, inode, page cache**까지 포함해서 들여다봐야 한다
- 데이터 전송/저장 비용은 압축(gzip/zstd)과 직렬화 포맷(avro/protobuf) 선택으로 줄이되, 스키마 호환성 관리가 뒤따라야 한다

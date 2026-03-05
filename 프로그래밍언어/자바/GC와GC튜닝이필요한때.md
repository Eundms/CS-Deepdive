- Serial GC 기준으로 설명
- JVM = hotspot VM 

1️⃣ 핵심 문서 : JDK25 공식 문서. HotSpot Virtual Machine Garbage Collection Tuning Guide
https://docs.oracle.com/en/java/javase/25/gctuning/

2️⃣ 참고 문서1 :JDK7 공식 문서. generational garbage collection 참고용
https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html

3️⃣ 참고 문서2 : 공식문서는 아니지만 공식문서가 레퍼런싱하는 사이트. ZGC 참고용
https://dev.java/learn/jvm/tool/garbage-collection/

-----
## Garbage Collector 
- 개발자가 직접 메모리를 할당받고 반납하는 관리 비용 제거 
    - OS로부터 메모리 영역을 확보하고 다시 반납하는 역할을 수행해줌
        - 애플리케이션에서는 객체 생성 등 메모리가 필요할 때마다 `GC가 확보한 메모리 영역`을 사용
        - GC는 OS로부터 할당 받은 메모리 영역 중에 애플리케이션이 실제로 사용하고 있는 영역과 그렇지 않은 영역을 파악 
        - 사용되지 않는 영역은 회수해서 어플리케이션이 다시 재사용할 수 있게 준비 
- GC로 인한 overhead 발생 

## GC 의 주요 테크닉
- 세대별 청소(generational scavenging)와 aging 기법을 사용해서 JVM Heap 내에서도 회수할만한 객체가 많이 있을 법한 영역에 집중한다 
- 살아있는 객체들은 한 곳에 모아서 최대한 연속된 free 영역을 확보하려고 노력한다.
- GC 동작 시 여러 스레드를 사용하여 병렬로 동작하고 오래 걸리면 백그라운드로 애플리케이션 코드와 동시에 실행될 수 있도록 한다.

## 언제 GC 선택과 튜닝이 중요한가?
- 기본적으로 자바실행환경이 알아서 애플리케이션이 실행되는 컴퓨터 시스템 특성을 보고 어떤 GC로 동작할지 선택한다 
- GC가 어느 정도의 빈도와 길이의 중단을 가지더라도 애플리케이션 성능이 잘나오면 별도의 GC 선택과 튜닝은 필요없다
- 기본 선택이 최선이 아닐 수 있으며, 스케일이 큰 애플리케이션, `데이터를 많이 쓰는 경우, 스레드를 많이 사용하는 경우, 높은 처리량을 요구하는 애플리케이션`의 경우에는 성능을 위해 별도의 GC 선택과 튜닝이 필요할 수 있다


- Amdahl's law : 어떤 문제에서 병렬을 통한 성능 향상은 해당 문제의 순차적인 부분에 의해 제한된다 
- 소규모 시스템에서는 별 문제가 안될 GC로 인한 성능 이슈가 대규모 시스템으로 확장될때는 주요 병목 지점이 될 수 있다
- 대규모 시스템에서는 GC 오버헤드를 조금만 낮춰도 성능 향상에 이점이 크기 때문에 GC 튜닝도 하는 것이 중요하고 가치있는 일이다
- Java는 4개의 가비지 컬렉터를 제공하는데 그 중에 serial GC를 제외하고는 모두 성능 향상을 위해 병렬로 동작한다 


## 언제 serial GC가 적절한가?
- 소규모 애플리케이션 : 100MB 이하의 Heap Size를 필요하는 애플리케이션 
- 사이즈가 큰 메모리와 멀티프로세서(멀티코어)를 갖춘 시스템에서 스케일이 크고 스레드를 헤비하게 쓰는 애플리케이션인 경우에는 serial GC가 1옵션이 될 수 없음
- 서버-클래스로 분류될 수 있는 머신에서 애플리케이션이 실행된다면 G1GC가 디폴트로 사용 


## Ergonomics 
- JVM이 주어진 환경에서 GC와 메모리 관련 설정을 스스로 최적화하는 것 (자동 최적화)
1. default selection : collector, heap size, GC thread 수, JIT compiler
2. 실행 중에 동적 조정 (behavior - based tuning)
    - 아래 두 가지 목표를 맞추기 위해 사용자가 설정한 목표를 모니터링하여 실행 중에 동적으로 조정
        - `Maximum Pause-Time goal(-XX:MaxGCPauseMillis)` 
        - `Throughout goal(-XX:GCTimeRatio)`
    - 두가지 목표가 모두 충족되는 동안 minimum footprint를 추구하며 동작

## JVM의 주요 default 설정
- garbage collector
    - `서버용(server-class) 머신`에서는 `G1GC(Garbage-First Garbage collector)`가 사용 그 외는 Serial Collector 사용 
        - 서버용 머신 판단 기준 : `두 개 이상의 프로세서`를 가지며 `RAM>=1792MB`인 경우 


- 서버 애플리케이션은 이 세개를 동일하게 맞춤
    - `initial heap size(JVM이 시작할 때 바로 확보하는 heap 크기)` : RAM의 1/64
    - `maximum heap size(JVM heap의 최대 크기)` : RAM 1/4
    - `minimum heap size(JVM heap의 최소 크기)` : 디폴트 값 설정이 복잡하게 결정됨 

- 최대 GC thread 수 : heap size와 이용 가능한 CPU resources에 의해 결정됨
- JIT compiler : C1과 C2를 모두 사용하는 tiered compiler 사용



### Maximum Pause-Time Goal
- pause time: GC로 인해 애플리케이션이 아예 멈추는 시간, `stop-the-world 시간`
- pause time이 아무리 길어도 `maximum pause-time goal`보다는 적어야 한다 
- maximum pause-time goal : `-XX:MaxGCPauseMillis=nnn밀리세컨드` 로 지정 
- GC는 pause time에 대한 가중평균과 분산을 계산해서 이 둘의 합을 maximum pause-time 으로 설정
- garbage collector는 heap size나 gc 관련 여러 파라미터들을 조정하여 `pause time을 nnn milliseconds` 보다 작게 유지하려고 시도
- maximum pause-time goal의 디폴트 값은 collector마다 다름 


### Throghput goal
- Throughput goal: GC time과 application time을 비교해서 특정 비율을 맞추도록 하는 것
    - GC time과 application time을 비교해서 특정 비율을 맞추도록 하는 것
- `-XX:GCTimeRatio=nnn`로 지정. `GC Time Ratio = 1/(1+nnn)`로 목표를 설정하는 것

## Minimum Footprint
- Footprint : 프로세스가 현재 사용중인 메모리의 크기. heap 크기가 전체 메모리 크기에 가장 영향력이 클 것임
- `throughput goal`과 `maximum pause-time goal`이 모두 충족되면 garbage collector는 heap 사이즈를 조금씩 줄인다 
    - 불필요하게 점유하는 메모리를 줄여서 다른 프로세스들도 메모리를 사용할 수 있게 해주기 위함
    - 두 goal 중에 하나를 충족하지 못할 때까지 줄여나감 (예외 없이 throughput goal이 먼저 깨짐)
- minimum과 maximum heap size는 각각 `-Xms=nnn`와 `-Xms=mmm` 옵션으로 지정할 수 있다


## GC 튜닝 가이드
1. throughput goal을 잡는다 
2. max heap size까지 써도 throughput goal을 달성하지 못하면 RAM 크기에 근접하게 max heap size를 잡는다 (단, swap은 안 일어나게)
3. 그럼에도 여전히 throughput goal 달성을 못하면 RAM 대비 목표 자체가 너무 높은 것
4. throughput goal을 달성했는데 pause time이 너무 길면 maximum pause-time goal을 지정한다.
5. maximum pause-time goal을 지정하면 throughput goal이 충족되지 않을 수 있다. 그러므로 적당한 타협이 필요하다
  - throughput goal과 maximum pause-time goal/minimum footprint는 trade off 관계다
  - 서버 애플리케이션처럼 -Xms = -Xmx로 지정하는 경우에는 이 튜닝 가이드의 효용성이 크진 않다.
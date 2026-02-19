# JVM
> JVM을 실행할 수 있는 환경이면 어디든지 .jar 파일을 동작시킬 수 있음

- .java - 컴파일(compile; javac) -> .class(bytecode) - 압축(jar) -> .jar  
( gradle, maven 같은 빌드 툴 사용 )  

- java -jar app.jar 실행시 JVM 위에서 동작 진행 

## JVM을 실행할 수 있는 환경은 어떻게 구성하나?
- JRE : jar파일로 JVM에서 실행만 
- JDK : 개발과 실행을 모두 하려면 (현재는 JDK만 공식 배포)

### JVM 실행 환경을 셋업하는 방법
- jdk 설치
- OS 패키지 팀에서 편의를 위해 제공하는 jre를 설치 
- 도커 환경에서 JRE스러운 이미지 사용
- jlink를 통해 나만의 실행 환경 구성 

/bin 아래에 있는 실행 파일들 

## 프로세스로 실행되는 건 jar 파일인가 java 실행 파일인가?
- /bin/java는 실행 파일이며 이게 프로세스로 실행됨
- 자바프로세스 = JVM 프로세스 
- JVM프로세스는 jar 내부의 .class 파일에 기술된 바이트코드들을 읽고 해석해서 대신 수행하거나 자주 사용되는 바이트코드들은 컴파일해서 기계어로 바로 실행될 수 있게 한다 

## /bin/java 실행 후 main 메서드 호출까지 주요 흐름
- CallJavaMainInNewThread
    - JavaMain(args) : 새로운 thread 생성
    - ThreadJavaMain : 새로운 thread의 start point
- JavaMain
    - InitalizeJVM : JVM 초기화 
        - JavaThread() : JVM(C++)에서 관리하는 JAVA Thread 객체 생성
    - LoadMainClass : Main 클래스 로딩
        - checkAndLoadMain
    - invokeStaticMainWithoutArgs : main 메서드 호출
        - GetStaticMethodId
        - CallStaticVoidMethod

- java.lang.Thread : Java Thread : OS thread = 1 : 1 : 1

java.lang.Thread 의 start() 내의 start0() native 

## JVM이 바이트코드를 실행하는 두 가지 흐름
- 인터프리터 방식 : 바이트코드를 한줄 한줄 해석해서 대신 실행
    - 바이트코드를 한줄 한줄 해석하면서 각각의 바이트코드 명령에 대해 어떻게 동작하라고 정의된 내부 코드를 실행하는 방식
    - java를 실행하면 우선 인터프리터로 바이트코드를 해석하며 대신 실행됨
- JIT 컴파일러 : Just-in-time 컴파일러
    - 바이트코드 중에 자주 호출되는 메서드나 loop 블록의 경우 아예 최적화해서 기계어 블락으로 컴파일하고 그 다음부턴 컴파일된 곳(code cache)으로 바로 호출 

> 인터프리터 방식으로 동작하다가 자주 호출되는 영역(hotspot)은 JIT compiler를 기계어로 컴파일하여 직접 실행되도록하는 방식의 VM을 hotspot VM이라고 한다.

- hotspot의 JIT comiler는 동작 방식이 tiered compilation 방식이다
    - level0 (interpreter) -> level1-3(c1-client compiler) -> level4 (c2-server compiler)
        - level이 높아질수록 더 최적화됨

## 클래스 파일들을 한번에 다 RAM에 로딩되나?
- 자바 표준 라이브러리는 /lib/modules라는 jimage포맷의 파일에 모두 포함되어 있음
- 우리가 개발한 자바 클래스들은 jar 파일에 있음 
- java 실행 시 `클래스 로더(classloader)`를 통해 실행에 필요한 최소한의 클래스만 klass 객체로만들어서 `metaspace`에 로딩 (java.lang.class 객체도 java heap에 생성)
- 로딩된 클래스를 바탕으로 바이트코드가 수행
- `lazy loading` : 바이트코드를 계속 수행하다가 아직 로딩되지 않은 클래스를 만나면 클래스 로더가 2ndaray storage에 가서 해당 클래스를 로딩한다 (new BestShop(), new ArrayList<>())
    - 웜업을 하는 이유 
- 클래스 로더 종류 
    - bootstrap classloader : 자바 표준 라이브러리를 담당 (/lib/modules)
    - application classsloader : 개발자가 작성한 클래스 담당 (.jar)
    - platform classloader : jdk확장할때 사용 
- 클래스 로더는 loading -> linking -> initialization 과정이 있음 


```markdown
JAVA_HOME
  - /bin : 각종 실행 파일
  - /lib 
    - modules : 자바 라이브러리 
    - lib*.so : 네이티브 라이브러리
    - /server
      - libjvm.so : JVM 엔진
```

## JVM 프로세스 메모리 구조
- virtual address space
    - kernel space | stack | native library | JVM heap | code cache | heap | data(+bss) | code(text)

- heap | data(+bss) | code(text) 
    >> JVM 내부 로직이 사용(C++로 개발된 코드들)    
        가령 이 heap에는 JavaThread 객체가 존재함 

- `JVM stack`
    - OS Thread마다 할당되는 stack 영역을 자바 thread들이 그대로 사용
    - `hostpot VM`은 java 스택과 native 스택을 합쳐서 사용 

- native library
    - libjvm.so, libnet.so, libjava.so.etc를 통해 os 시스템 콜도 호출 가능

- `Metaspace(method area)`
   - kclass 객체 : 각종 클래스 메타 정보 (필드, 메서드 정보, 상속 관계)
   - 메서드 바이트코드 
     - Runtime constant pool 
     - `드물게 GC 발생 (fullGC)`

- `JVM heap`
    - 자바 객체와 자바 배열이 여기에 상주
    - java.lang.Thread 객체나 java.lang.Class 객체도 여기
    - 우리가 배울 GC는 이부분에 대한 내용임
    - String pool도 여기 

- Code cache
    - JIT 컴파일러에 의해 최적화되어 기계어로 변환된 코드들이 상주 



## 1. synchronized 키워드
- 가장 기본적인 동기화 메커니즘
- 메서드나 블록 단위로 동기화
- `재진입 가능` (같은 스레드가 여러 번 획득 가능)
- 단순하지만 제어가 제한적

```java
// 메서드 동기화
public synchronized void method() {
    // 동기화된 코드
}

// 블록 동기화
synchronized (object) {
    // 동기화된 코드
}
```

## 2. ReentrantLock
- `synchronized`보다 더 유연한 락 구현
- 주요 특징:
	- 재진입 가능
	-  공정성(fairness) 옵션 제공
	-  타임아웃 설정 가능
	- 조건 변수(Condition) 지원
	- 락 획득 시도 횟수 추적 가능

```java
ReentrantLock lock = new ReentrantLock();

// 기본 사용
lock.lock();
try {
    // 동기화된 코드
} finally {
    lock.unlock();
}

// 타임아웃 설정
if (lock.tryLock(timeout, unit)) {
    try {
        // 동기화된 코드
    } finally {
        lock.unlock();
    }
}

// 공정한 락
ReentrantLock fairLock = new ReentrantLock(true);
```

## 3. ReadWriteLock
- `읽기/쓰기 작업을 분리`하여 동시성 제어
- 여러 스레드가 동시에 읽기 가능
- 쓰기 작업 시에는 배타적 접근

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// 읽기 작업
readLock.lock();
try {
    // 여러 스레드가 동시에 읽기 가능
} finally {
    readLock.unlock();
}

// 쓰기 작업
writeLock.lock();
try {
    // 한 스레드만 쓰기 가능
} finally {
    writeLock.unlock();
}
```

## 4. StampedLock
- `낙관적 읽기 락` 지원
- ReadWriteLock보다 더 나은 성능
- `낙관적 읽기`, `비관적 읽기`, `쓰기 락` 제공

```java
StampedLock stampedLock = new StampedLock();

// 낙관적 읽기
long stamp = stampedLock.tryOptimisticRead();
try {
    // 읽기 작업
    if (!stampedLock.validate(stamp)) {
        // 낙관적 읽기 실패 시 비관적 읽기로 전환
        stamp = stampedLock.readLock();
        try {
            // 읽기 작업
        } finally {
            stampedLock.unlockRead(stamp);
        }
    }
}

// 쓰기
long stamp = stampedLock.writeLock();
try {
    // 쓰기 작업
} finally {
    stampedLock.unlockWrite(stamp);
}
```

## 5. Semaphore
- 특정 리소스에 대한 접근을 제한
- 동시에 접근 가능한 스레드 수 제어

```java
// 최대 3개의 스레드만 동시 접근 가능
Semaphore semaphore = new Semaphore(3);

try {
    semaphore.acquire();
    try {
        // 제한된 리소스 사용
    } finally {
        semaphore.release();
    }
} catch (InterruptedException e) {
    // 인터럽트 처리
}
```

## 각 락의 사용 사례

### ReentrantLock
- 단순한 배타적 접근 제어가 필요한 경우
- 타임아웃을 통한 데드락 방지가 필요한 경우
- 조건 변수를 통한 세밀한 제어가 필요한 경우

### ReadWriteLock
- 읽기 작업이 많은 경우
- 캐시 구현
- 데이터베이스 접근

### StampedLock
- 고성능이 필요한 읽기 작업
- 낙관적 읽기가 가능한 경우
- ReadWriteLock보다 더 나은 성능이 필요한 경우

### Semaphore
- 리소스 풀 구현
- 동시 접근 수 제한
- 제한된 리소스 관리


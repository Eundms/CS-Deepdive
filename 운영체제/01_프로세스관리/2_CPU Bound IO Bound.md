# CPU Bound vs I/O Bound 프로세스
- `CPU Bound 프로세스`
> CPU를 장시간 점유 -> 다른 프로세스가 실행될 기회 제한
- 멀티 코어 시스템에서 병렬처리 
- 타임 슬라이스 기반 스케줄링에서 비효율적임
> 멀티 코어 CPU에서 병렬 처리 (멀티프로세싱)

-  `I/O Bound 프로세스`
> CPU보다 I/O 작업(디스크, 네트워크 요청 등)이 많은 경우 Waiting 상태에서 머무는 시간이 많음
- 대부분의 시간 동안 I/O 작업을 기다림 → CPU 자원이 유휴 상태가 될 가능성이 높음
- 많은 프로세스가 동시에 실행되면 I/O 병목 현상 발생 가능  
> 비동기 혹은 멀티스레딩 기법 활용, 캐시와 버퍼 활용 

## 듀얼 코어 CPU에서 동작할 CPU Bound 프로그램을 구현한다면 몇 개의 스레드를 쓰는 것이 좋을까?
- `CPU Bound 프로그램에서 적절한 스레드의 수 = number of CPUs + 1`

## API 서버가 thread per request 방식이라면?
- 몇 개의 스레드들을 미리 만들어 놓을지 여러 상황을 고려해서 결정하는 것이 필요 

## CPU Bound, I/O Bound 연산 처리 예시 
- CPU Bound
```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> heavyComputation());
executor.shutdown();
```

- I/O Bound
```java
CompletableFuture.runAsync(() -> fetchDataFromDatabase());
```



### ForkJoinPool 이용해 대규모 데이터 연산 병렬 처리 
- 작업을 쪼개 compute()에서 계산 후 병합
- fork()와 join()을 이용해 병렬 처리 최적화 

```java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;

class SumTask extends RecursiveTask<Long> {
    private final int start, end;
    private final int THRESHOLD = 1000; // 임계값
    private final int[] numbers;

    public SumTask(int[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if ((end - start) <= THRESHOLD) {
            // 작은 작업이면 직접 계산
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += numbers[i];
            }
            return sum;
        } else {
            // 큰 작업이면 분할
            int mid = (start + end) / 2;
            SumTask leftTask = new SumTask(numbers, start, mid);
            SumTask rightTask = new SumTask(numbers, mid, end);

            leftTask.fork();  // 비동기 실행
            long rightResult = rightTask.compute();
            long leftResult = leftTask.join(); // 왼쪽 작업 완료 대기

            return leftResult + rightResult;
        }
    }
}

public class ForkJoinExample {
    public static void main(String[] args) {
        int[] numbers = new int[10_000_000];
        for (int i = 0; i < numbers.length; i++) numbers[i] = i;

        ForkJoinPool pool = new ForkJoinPool();
        long result = pool.invoke(new SumTask(numbers, 0, numbers.length));

        System.out.println("총 합: " + result);
    }
}
```

### ExecutorService 멀티 프로세싱을 위한 쓰레드 풀
- ExecutorService는 고정된 개수의 쓰레드를 활용하여 병렬 작업 처리 
- 적절한 스레드 개수를 유지하여 CPU 과부하 유지 
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(4); // 4개의 쓰레드 풀 생성

        for (int i = 0; i < 10; i++) {
            int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " 실행 - " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}
```

### 비동기 멀티 스레딩 CompletableFuture 활용 
- CompletableFuture : 비동기 실행 및 콜백 처리 기능 
- 기존의 Future 대비 콜백 등록이 가능하고 체이닝이 쉬움  
- API 호출, 네트워크 요청 

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureChaining {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("Step 1: 데이터 로드");
            return "데이터";
        }).thenApply(data -> {
            System.out.println("Step 2: 데이터 변환");
            return data + " 변환 완료";
        }).thenAccept(result -> {
            System.out.println("Step 3: 결과 출력 -> " + result);
        });

        future.get(); // 최종 결과 대기
    }
}
```
Proxy 객체 지나고 접근하는 패턴 
-> 중요한 객체에 대한 접근제어, 초기화 지연, 로깅 또는 캐싱에 적용 가능

![[Pasted image 20250530175623.png]]




[[데코레이터(Decorator) 패턴]]



특정 객체에 대한 접근을 제어하거나 기능을 추가할 수 있는 패턴
- 초기화 지연, 접근 제어, 로깅, 캐싱 등 다양하게 응용해 사용할 수 있다 


장점 
- 기존 코드 변경하지 않고 새로운 기능 추가 가능
- 기존 코드가 해야 하는 일만 유지할 수 있음
- 기능 추가 및 초기화 지연 등으로 다양하게 활용 가능

단점
- 코드의 복잡도가 증가 


---

- before
```java
public class GameService {  
    public void startGame() {  
       System.out.println("이 자리에 오신 여러분을 진심으로 환영합니다.");  
    }  
}

public class Client {  
    public static void main(String[] args) {  
       GameService gameService = new GameService();  
       gameService.startGame();  
    }  
}
```


- after
```java
public class Client {  
    public static void main(String[] args) {  
       // defaultGameservice를 proxy를 거쳐서 사용  
       GameService gameService = new GameServiceProxy(new DefaultGameService());  
       gameService.startGame();  
    }  
}


public class GameServiceProxy implements GameService {  
    private GameService gameService;  
  
    public GameServiceProxy(GameService gameService) {  
       this.gameService = gameService;  
    }  
  
    @Override  
    public void startGame() {  
       long before = System.currentTimeMillis();  
       gameService.startGame();  
       System.out.println(System.currentTimeMillis() - before);  
    }  
}


public class DefaultGameService implements GameService {  
  
    @Override  
    public void startGame(){  
       System.out.println("이 자리에 오신 여러분을 진심으로 환영합니다.");  
       try {  
          Thread.sleep(1000L);  
       } catch (Exception e) {  
  
       }  
    }  
}

public interface GameService {  
    void startGame();  
}

```


- 느린 초기화 // 이런식으로 캐싱도 할 수 있음 
```java
public class GameServiceProxy implements GameService {
    private GameService gameService;
    @Override
    public void startGame() {
	    long before = System.currentTimeMillis();
	    if (this.gameService == null) { // 느린 초기화 
	        this.gameService = new DefaultGameService();
	    }
	    
	    gameService.startGame();
	    System.out.println(System.currentTimeMillis() - before);
    }
}
```



---
출처
https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%ED%8C%A8%ED%84%B4
코딩으로 학습하는 GoF 디자인 패턴 
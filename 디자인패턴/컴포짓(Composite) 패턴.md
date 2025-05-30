> Tree 구조 


전체와 개별 객체를 동일하게 처리할 수 있는 패턴
클라이언트 입장에서는 전체나 부분이나 모두 동일한 컴포넌트로 인식할 수 있는 계층 구조 

![[Pasted image 20250530150703.png]]
- 위키백과 : 컴포지트 패턴 

```java
public class Bag implements Component {  
    private List<Component> components;  
  
    public Bag() {  
       components = new ArrayList<>();  
    }  
  
    public void add(Component component) {  
       components.add(component);  
    }  
  
    public List<Component> getComponents() {  
       return components;  
    }  
  
    @Override  
    public int getPrice() {  
       return components.stream().mapToInt(Component::getPrice).sum();  
    }  
}

public interface Component {  
    int getPrice();  
  
}

public class Item implements Component {  
    private String name;  
    private int price;  
    public Item(String name, int price) {  
       this.name = name;  
       this.price = price;  
    }  
  
    @Override  
    public int getPrice() {  
       return this.price;  
    }  
}
```



- 장점 / 단점
	- 트리구조로 표현할 수 있다 -> 런타임에 타입을 체크해야 하는 경우도 있음
	- 새로운 종류의 컴포넌트를 추가할 수 있다 




---
출처
https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8F%AC%EC%A7%80%ED%8A%B8_%ED%8C%A8%ED%84%B4
코딩으로 학습하는 GoF 디자인 패턴 
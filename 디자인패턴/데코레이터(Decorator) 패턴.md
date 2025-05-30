기존 코드를 변경하지 않고 부가 기능을 추가하는 패턴
- 상속이 아닌 위임을 사용해서 보다 유연하게(런타임에) 부가 기능을 추가하는 것도 가능

![[Pasted image 20250530153543.png]]
- 위키 

장점
- 새로운 클래스를 만들지 않고 기존 기능을 조합할 수 있다
	- 단일 책임 원칙 
- 런타임에 동적으로 기능을 변경할 수 있다

단점
- 데코레이터를 조합하는 코드가 복잡할 수 있다 


---


- before : 상속을 이용한다면 CommentService를 상속받은 여러 기능을 가진 CommentService 사용
```java
public class Client {
	private CommentService commentService;
	public Client(CommentService commentService) {
		this.commentService = commentService;
	}
	private void writeComment(String comment) {
		commentService.addComment(comment);
	}
	public static void main(String[] args) {
		Client client = new Client(new CommentService());
		client.writeComment("오징어게임");
		client.writeComment("보는게 하는 거보다 재미있을 수 가 없지...");
	}
}
```

- after 
```java
public interface CommentService {  
    void addComment(String comment);  
}

public class Client {  
    private CommentService commentService;  
    public Client(CommentService commentService) {  
       this.commentService = commentService;  
    }  
    public void writeComment(String comment) {  
       commentService.addComment(comment);  
    }  
  
}

public class CommentDecorator implements CommentService {  
    private CommentService commentService; // 기존 서비스를 받음 
  
    public CommentDecorator(CommentService commentService) {  
       this.commentService = commentService;  
    }  
  
    @Override  
    public void addComment(String comment) {  
       commentService.addComment(comment);   
    }  
}


public class TrimmingCommentDecorator extends CommentDecorator {  
  
    public TrimmingCommentDecorator(CommentService commentService) {  
       super(commentService);  
    }  
  
    @Override  
    public void addComment(String comment) {  
       super.addComment(trim(comment));  // 부모 호출할때 trim(comment)해서 보냄
    }  
  
    private String trim(String comment) {  
       return comment.replace("...", "");  
    }  
}


public class App {  
    private static boolean enabledSpamFilter = true, enabledTrimming = true;  
  
    public static void main(String[] args) {  
       CommentService commentService = new DefaultCommentService();  
  
       if(enabledSpamFilter) { // 한번 감싼다  
          commentService = new SpamFilteringCommentDecorator(commentService);  
       }  
  
       if(enabledTrimming) {  
          commentService = new TrimmingCommentDecorator(commentService);  
       }  
  
       // 클라이언트 측면에서는 동일함  
       Client client = new Client(commentService);  
       client.writeComment("오징어게임");  
       client.writeComment("http://whiteship.me");  
  
    }  
}
```


---
출처
https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0_%ED%8C%A8%ED%84%B4
코딩으로 학습하는 GoF 디자인 패턴 
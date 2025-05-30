

## 아래 코드는 멀티 스레드 환경에서 안전하지 않다
```java
public class Settings {  
    private static Settings instance;  
    private Settings() {  
  
    }  
    public static Settings getInstance() {  
       if(instance == null) {  
          instance = new Settings();  // 여기 2개 new 될 수 있음
       }  
       return instance;  
    }  
}

public class App {  
    public static void main(String[] args) {  
       Settings settings = Settings.getInstance();  
       System.out.println(settings == Settings.getInstance());  
    }  
}
```


### sychronized 키워드 : 동기화로 인해 부가적인 성능 부하 
```java
public class Settings {  
    private static Settings instance;  
    private Settings() {  
  
    }  
    public static synchronized Settings getInstance() {  
       if(instance == null) {  
          instance = new Settings();  
       }  
       return instance;  
    }  
}
```

### eager initialization : 스레드 세이프함 -> 클래스 로딩 시점에 초기화 되고 미리 만들어두기 때문에 ok
	- 하지만 미리 초기화 하는게 싫으면 다른 방안 생각해보자 
```java
public class Settings {  
    private static final Settings INSTANCE = new Settings();  
    private Settings() {  
  
    }  
    public static Settings getInstance() {  
       return INSTANCE;  
    }  
}
```

### double checked locking 사용하기 
	- 호출할 때마다 synchronized가 걸리지 않기에 좀 나음 
	- 인스턴스를 필요한 시점에 만들수 있음
``` java
public class Settings {  
    private static volatile Settings instance;  // java 1.5부터 동작
  
    private Settings() {}  
  
    public static Settings getInstance() {  // 호출할 때마다 synchronized가 걸리지 않기에 좀 나음 
       if (instance == null) {  
          synchronized (Settings.class) {  
             if (instance == null) {  
                instance = new Settings();  
             }  
          }  
       }  
       return instance;  
    }  
}
```


### static inner 클래스 사용하기 (권장하는 방법 중 하나 )
```java
public class Settings {  
    private Settings() {}  
  
    private static class SettingsHolder {  
       private static final Settings INSTANCE = new Settings();  
    }  
  
    public static Settings getInstance() {  
       return SettingsHolder.INSTANCE;  
    }  
}
```


## 이러한 것들을  깨뜨리는 코딩 방식 

### 1.  Reflaction
```java
public class App {  
    public static void main(String[] args) throws Exception {  
       Settings settings = Settings.getInstance();  
       Constructor<Settings> constructor = Settings.class.getDeclaredConstructor();  
       constructor.setAccessible(true);  
       Settings settings1 = constructor.newInstance();  
       System.out.println(settings == settings1);
    }
}
```

### 2.  직렬화 & 역직렬화 
```java
public class App {
	public static void main(String[] args) throws Exception {  
	    Settings settings = Settings.getInstance();  
	    try (ObjectOutput output = new ObjectOutputStream(new FileOutputStream("settings.obj"))) {  
	       output.writeObject(settings);  
	    }  
	    Settings settings1 = null;  
	    try (ObjectInput input = new ObjectInputStream(new FileInputStream("settings.obj"))) {  
	       settings1 = (Settings) input.readObject();  
	    }  
	    System.out.println(settings1 == settings);
	}
}
```
- 동일한 객체 얻은 방법
```java
public class Settings implements Serializable {  
    private Settings() {}  
  
    private static class SettingsHolder {  
       private static final Settings INSTANCE = new Settings();  
    }  
  
    public static Settings getInstance() {  
       return SettingsHolder.INSTANCE;  
    }  
      
    protected Object readResolve() {   // readResolve 재정의 
       return getInstance();  
    }  
}
```


#### => Reflection에  안전한 방법 : enum  (권장하는 방안)
```java
public enum Settings { // serializable 상속 중 
    INSTANCE;
}
```

- `enum` 클래스는 **컴파일러와 JVM이 강하게 제어**합니다.
- `Class.getDeclaredConstructors()`를 통해 생성자를 얻으려 해도 예외를 발생시킵니다.
- `enum`은 기본적으로 `Serializable`을 구현하고, `readResolve()`를 사용하지 않아도 **항상 같은 인스턴스를 반환**합니다.
- JVM이 enum 클래스의 로딩을 thread-safe하게 처리해 줍니다.
- synchronized 없이도 안전하게 Singleton 보장

- `INSTANCE`는 자동으로 `static final`로 선언되고, 생성자도 자동으로 **한 번만 호출되도록** 보장된다
- 우리가 `new Settings()`를 호출하지 않아도, **JVM이 enum 생성자 호출을 처리**한다



---
출처

코딩으로 학습하는 GoF 디자인 패턴 
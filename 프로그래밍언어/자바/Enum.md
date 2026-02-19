```java
public enum ProductType {
    DISICOUNT_10000 {
        public int discount(int amount) {
            return amount - 10000;
        }
    }, 
    PERCENT_10 {
        public int discount(int amount) {
            return (int)(amaount * 0.9);
        }
    }
    public abstract int discount(int amount);
}
```

```java
public class ProductService {
    public int applyDiscount(ProductType productType, int price) {
        return productType.discount(price);
    }
}
```


- 정책별 구현 클래스 정의 
```java
@Component("DISCOUNT_1000")
public class Discount1000Policy implements DiscountPolicy {
    @Override
    public int discount(int amount) {
        return amount - 1000;
    }
}

@Component("PERCENT_10")
public class Percent10Policy implements DiscountPolicy {
    @Override
    public int discount(int amount) {
        return (int)(amount * 0.9);
    }
}
```

# book-object
Object (조영호) 도서를 읽고 정리하는 레포

# 목차  

[new는 해롭다](https://github.com/yeoseon/book__object#new%EB%8A%94-%ED%95%B4%EB%A1%AD%EB%8B%A4)  
[가끔은 생성해도 무방하다](https://github.com/yeoseon/book__object#%EA%B0%80%EB%81%94%EC%9D%84-%EC%83%9D%EC%84%B1%ED%95%B4%EB%8F%84-%EB%AC%B4%EB%B0%A9%ED%95%98)  
[표준 클래스에 대한 의존은 해롭지 않다](https://github.com/yeoseon/book__object#%ED%91%9C%EC%A4%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%98%EC%A1%B4%EC%9D%80-%ED%95%B4%EB%A1%AD%EC%A7%80-%EC%95%8A%EB%8B%A4)  

# 08 의존성 관리하기  

## 01. 의존성 이해하기  

## 02. 유연한 설계  

### ```new```는 해롭다  

#### ```new```가 해로운 이유 (결합도 측면)  

* ```new``` 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야하여 의존도가 높아진다.  
* 구체 클래스의 어떤 인자를 이용해야하는 지까지 알아야 한다.  

> 결합도가 높으면 변경에 영향을 받기가 쉽다  

#### 해결 방법 : 사용과 생성의 책임 분리   

**출발: 객체를 생성하는 책임을 객체 내부가 아니라 클라이언트로 옮긴다.**  
**1. 사용과 생성의 책임을 분리하고,**  
**2. 의존성을 생성자에 명시적으로 드러내고,**  
**3. 구체 클래스가 아닌 추상 클래스에 의존하게 한다.**  

인스턴스를 **생성하는 로직**과 생성된 인스턴스를 **사용하는 로직**을 **분리**하자.  
사용하는 클래스는, **외부로부터 이미 생성된 인스턴스를 주입**받는 방식으로 구현한다.  
인스턴스를 생성하는 책임은 사용하는 클래스(예. ```Movie```)의 클라이언트가 수행하게 된다.  

```
public class Movie { 
    ...
    private DiscountPolicy discountPolicy;  

    public Movid(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
    ...
    this.discountPolicy = discountPolicy;    
    }
}
```
* 구체 클래스를 사용하지 않고 추상 클래스를 인자로 받아들일 수 있어 설계가 유연해진다.  
* **올바른 객체가 올바른 책임을 수행하게 하자**  

### 가끔을 생성해도 무방하다 

협력하는 기본 객체를 설정하고 싶은 경우, 클래스 내에서 직접 생성하는 방식이 유용할 수 있다.  

(예) ```Movie```가 대부분의 경우에는 ```AmountDiscountPolicy```의 인스턴스만 협력하고 가끔씩만 ```PercentDiscountPolicy```의 인스턴스와 협력한다고 할 때, 인스턴스 생성하는 책임을 클라이언트로 옮긴다면, 중복 코드가 늘어날 가능성이 있다.  

#### 기본 생성자를 추가하고, 체이닝하자.  

```
public class Movie {
    private DiscountPolicy discountPolicy;  

    public Movid(String title, Duration runningTime) {
        this(title, runningTime, fee, new AountDiscountPolicy(...));
    }

    public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
        ...
        this.discountPolicy = discountPolicy;
    }
}
```

* 추가된 생성자 안에서 필요한 인스턴스를 생성하고 있다.  
* 그리고 첫번째 생성자를 호출하여 **체이닝**하고 있다.  

```
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        return calculateMovieFee(screening, new AmountDiscountPolicy(...)));
    }

    public Money calculateMovieFee(Screening screening, DiscountPolicy discountPolicy) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

* 위와 같이 메서드를 오버로딩하는 방법으로도 사용할 수 있다.  

#### 트레이드 오프 : 결합도와 사용성  

구체 클래스에 의존하게 되더라도 클래스의 사용성이 더 중요하면 결합도를 높힐 수 있다.  
**가급적 구체 클래스에 대한 의존성을 제거할 수 있는 방향을 추구하는 것이 좋다.**  
두 마리의 토끼를 모두 잡도록 노력해보자.  

### 표준 클래스에 대한 의존은 해롭지 않다.  

#### 의존성이 불편한 이유 : 변경에 대한 영향을 암시하기 때문에  

변경된 확률이 거의 없는 클래스라면 의존성이 문제가 되지 않는다.  

(예) JDK의 표준 컬랙션 라이브러리에 속하는 ```ArrayList```의 경우는 수정될 확률이 0에 가깝기 때문에 인스턴스를 직접 생성해도 관계 없다.  
```
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>(); 
}
```

#### 직접 생성하더라도, 가능한 추상적인 타입을 사용하자.  

**의존성에 의한 영향이 적은 경우에도 추상화에 의존하고, 의존성을 명시적으로 드러내는 습관을 가지자**  

```
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();
    
    public void switchConditions(List<DiscountCondition> conditions) {
        this.conditions = conditions;
    }
}
```
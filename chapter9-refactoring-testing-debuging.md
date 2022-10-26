

# 챕터 9 - 리팩토링, 테스팅, 디버깅

---

람다, 메서드 참조, 스트림 등을 이용해 리팩토링하면, 코드의 가독성과 유연성을 개선할 수 있다.



코드 가독성을 개선하는 방법 3가지

- 익명클래스를 람다 표현식으로 리팩토링
- 람다 표현식을 메서드 참조로 리팩토링
- 명령형 데이터 처리를 스트림으로 리팩토링

하나씩 살펴보자.


## 익명 클래스를 람다 표현식으로 리팩토링

함수형 인터페이스를 구현하는 익명 클래스는 람다 표현식으로 리팩토링 할 수 있다.

```
//함수형 인터페이스를 익명 클래스로 구현
Runnable r1 = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello");
        }
    };
    
// 람다 표현식을 이용해 리팩토링
Runnable r2 = () -> System.out.println("Hello");
```

물론 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다. 인스턴스 타입을 명시하는 익명 클래스와 달리 람다의 형식은 맥락(Context)에 따라 달라질 수 있다. 따라서 익명 클래스에서 사용하는 방식 그대로 변수나, This를 사용할 수 없다. 아래 코드와 같이 람다 표현식 안에서 지역 변수를 사용할 수 없다.

```
int a = 10;

Runnable r1 = new Runnable() {
    int a = 20;    
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

Runnable r2 = () -> {
    int a = 20; // compile error
    System.out.println("Hello");
};
```



## 람다 표현식을 메서드 참조로 리팩토링


메서드 참조를 람다 표현식에 활용하면 가독성과 코드의 의도를 전달하는데 도움이 된다.

```
// 스트림을 이용해 모든 요소를 탐색 후 분류
Map<CaloricLevel, List<Dish>> dishByCaloricLevel = menu.stream()
                .collect(groupingBy(dish -> {
                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                    else return CaloricLevel.FAT;
                }));


public CaloricLevel getCaloricLevel() {
    if (this.calories <= 400) return CaloricLevel.DIET;
    else if (this.calories <= 700) return CaloricLevel.NORMAL;
    else return CaloricLevel.FAT;
}

// 메서드 참조로 선언형 구현이 가능
Map<CaloricLevel, List<Dish>> dishByCaloricLevel = menu.stream()
                .collect(groupingBy(Dish::getCaloricLevel));
```

정적 헬퍼 메서드가 있다면 코드 가독성을 높일 수 있다.
```
inventory.sort((Apple a1, Apple a2) -> a1. getWeight().compareTo(a2.getWeight()));

inventory.sort(comparing(Apple::getWeight))
```



## 명령형 데이터 처리를 스트림으로 리팩토링하기


스트림을 이용한 리팩토링이 갖는 중요한 의미가 있다. 명령형 데이터 처리를 선언형으로 변환할 수 있다는 것이다. 아래 코드는 직접 반복문을 이용해 요소를 분류하고, 수집한다. 이는 스트림을 이용한 리팩토링이 가능하다

```
// 명령형 데이터 처리
List<String> dishNames = new ArrayList<>();
for (Dish dish : menu) {
  if (dish.getCalories() > 300) {
    dishNames.add(dish.getName());
  }
}

// 스트림을 이용해 선언형으로 리팩토링
menu.stream()
  .filter(d -> d.getCalories() > 300)
  .map(Dish::getName)
  .collect(toList());
```



---

# 람다로 객체지향 디자인 패턴 리팩토링


흔히 사용하는 디자인 패턴에 람다 표현식을 활용할 수 있다. 람다 표현식으로 기존의 객체지향 디자인 패턴을 어떻게 리팩토링 할 수 있는지 살펴보자

- 전략(Strategy)
- 템플릿 메서드(Template Method)
- 옵저버(Observer)
- 의무 체인(Chain of Responsibility)
- 팩토리(Factory)



## 전략 패턴

전략 패턴은 공통된 알고리즘으로 실행가능한 다수의 전략을 런타임 환경에서 선택하는 패턴이다. 쉽게 말해 공통된 로직 안에서 클라이언트에 따라 A 또는 B의 전략이 선택된다. 아래 코드를 통해 두 가지 전략이 선택되는 과정을 살펴보자. 인수로 전달되는 클라이언트에 따라 전략이 선택된다.

```
@FunctionalInterface
public interface ValidationStrategy {
    boolean execute(String s);
}

public class IsAllLowerCase implements ValidationStrategy {
    @Override
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}

public class isNumeric implements ValidationStrategy {
    @Override
    public boolean execute(String s) {
        return s.matches("\d+");
    }
}

public class Validator {
    private final ValidationStrategy validationStrategy;

    public Validator(ValidationStrategy validationStrategy) {
        this.validationStrategy = validationStrategy;
    }
    
    public boolean validate(String s) {
        return validationStrategy.execute(s);
    }
}

Validator numericValidator = new Validator(new isNumeric());
numericValidator.validate("aaa");

Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
lowerCaseValidator.validate("bbbb");
```

전략 패턴을 람다로 사용해 구현할 수 있다.
```
Validator lowerCaseValidator2 = new Validator((String s) -> s.matches("[a-z]+"));
lowerCaseValidator2.validate("bbbb");

Validator numericValidator2 = new Validator((String s) -> s.matches("\d+"));
numericValidator2.validate("1234");
```


ValidationStreategy는 함수형 인터페이스이며 Predicate과 같은 함수 디스크립터를 갖고 있다. 이러한 경우 전략에 해당하는 새로운 클래스를 매번 만드는 것 보다 람다 표현식을 이용하는 것이 더 나을 수 있다. 람다 표현식을 사용하면 전략을 캡슐화 하는데도 도움이 된다.



## 템플릿 메서드


템플릿 메서드는 추상 메서드를 이용해 특정 기능의 일부를 변경하여 사용하는 패턴이다. 람다 표현식을 사용하여 동일한 기능을 사용할 수 있다
```
// 템플릿 메서드 패턴
abstract class OnlineBanking {
  public void processCustomer(int id) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy(c);
  }

  abstract void makeCustomerHappay(Customer c);
}

// 람다 표현식 사용
public void processCustomer(int id, Cusumer<Customer> makeCustomerHappy) {
  Customer c = Database.getCustomerWithId(id);
  makeCustomerHappy.accept(c);
}

new OnlineBankingLambda().processCustomer(1337, (Customer c) -> print("hello" + c.getName()));
```



## 옵져버 패턴

특정한 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 알림을 보내는 패턴을 의미한다. notifyObserver() -> notify() <- ObserverA, ObserverB 와 같은 구조를 통해 Subject에 해당하는 객체가 Observer에 자동으로 알림을 보낸다.
```
interface NotiObserver {
  void notify(String tweet);
}

public class NyTimes implements NotiObserver {
    @Override
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("monety")) {
            System.out.println("Breaking news in NY ! " + tweet);
        }
    }
}

public class Guardian implements NotiObserver {
    @Override
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("queen")) {
            System.out.println("Yet more new from London .. " + tweet);
        }
    }
}

public class LeMonde implements NotiObserver {
    @Override
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("wine")) {
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}


public interface NotiSubject {
    void registerObserver(NotiObserver o);
    void notifyObservers(String tweet);
}

public class Feed implements NotiSubject {
    private final List<NotiObserver> observers = new ArrayList<>();
    @Override
    public void registerObserver(NotiObserver o) {
        observers.add(o);
    }

    @Override
    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}

Feed f = new Feed();
f.registerObserver(new NyTimes());
f.registerObserver(new LeMonde());
f.registerObserver(new Guardian());
f.notifyObservers("The Queen ...")
```

람다 표현식을 사용하면 아래와 같이 이벤트를 전달할 수 있다. 하지만 항상 람다 표현식을 이용한 리팩토링이 정답은 아니다. 맥락에 따라 기존 방식을 사용하는 것이 나을 수 있다.

```
Feed feed = new Feed();
feed.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("money")) {
        System.out.println("Breaking news in NY ! " + tweet);
    }
});
```



## 의무 체인


연속적인 작업 처리를 체이닝 방식으로 구현하는 패턴이다. 선두 객체의 작업이 완료됨과 동시에 다음 객체에 작업 명령을 전달한다.

```
// 일반적으로 작업 처리 추상 클래스를 이용해 체이닝을 구현한다
public abstract class ProcessingObject<T> {
    protected ProcessingObject<T> successor;
    
    public void setSuccessor(ProcessingObject<T> successor) {
        this.successor = successor;
    }
    
    public T handle(T input) {
        T r = hadleWork(input);
        if (successor != null) {
            return successor.handle(r);
        }
        return r;
    }
    
    abstract protected T hadleWork(T input);
}


public class HandleTextProcessing extends ProcessingObject<String> {
    @Override
    protected String hadleWork(String input) {
        return "From Raoul, Mario and Alan : " + input;
    }
}

public class SpellCheckProcessing extends ProcessingObject<String> {
    @Override
    protected String hadleWork(String input) {
        return input.replaceAll("labda", "lambda");
    }
}


ProcessingObject<String> p1 = new HandleTextProcessing();
ProcessingObject<String> p2 = new SpellCheckProcessing();
p1.setSuccessor(p2);
p1.handle("Aren't ladbas really sexy?");
```

이러한 패턴은 함수 체이닝 방식과 비슷하다. compose나 andThen을 이용하면 비슷한 조합을 만들 수 있다
```
UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan : " + text;
UnaryOperator<String> spellCheckProcessing = (String text) -> text.replaceAll("labda", "lambda");
Function<String, String> pipeline = headerProcessing.andThen(spellCheckProcessing);
pipeline.apply("Aren't ladbas really sexy?");
```



## 팩토리


인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들기 위해 사용하는 패턴이다.

```
public class ProductFactory {
    public static Product createProduct(String name) {
        switch (name) {
            case "loan" : return new Loan();
            case "stock" : return new Stock();
            case "bond" : return new Bond();
            default: throw new RuntimeException("");
        }
    }
}

Product p = ProductFactory.createProduct("loan");
```

생성자를 메서드 참조 방식으로 사용해 팩토리 메서드를 구현할 수 있다. 하지만, 앞서 옵저버 패턴에서 언급한 바와 같이 맥락에 따라 일반적인 방식이 더 나을 수 있다. 팩토리 패턴의 경우 파라미터가 많다면 더 복잡해질 수 있다.
```
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();

final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
  map.put("loan", Loan::new);
  map.put("stock", Stock::new);
  map.put("bond", Bond::new);
}

public static Product createProduct(String name) {
  Supplier<Product> p = map.get(name);
  if (p != null) {
    return p.get();
  }
  ...
}
```

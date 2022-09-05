# [Chapter 2] 동작 파라미터화 코드 전달하기
<br>

## Summary
* 동작 파라미터화는 메서드 내부적으로 다양한 동작을 유연하게 대응 할 수 있도록 코드를 메서드 인수로 전달한다.
* 불필요하거나 반복되는 코드를 줄이고 변화하는 요구사항에 유연하게 대응을할 수 있어 엔지니어링 비용을 줄일 수 있다.
* 코드 전달 기법(람다)를 통해 메서드의 인수를 전달할 수 있다. 익명 클래스로도 구현 할 수 있지만 람다를 이용하면 코드가 보다 간결해진다.
* 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다.


## 예제로 보는 단계적 코드 개선
### 1. if 조건을 통한 필터링
```java
enum Color { RED, GREEN }
```
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
 List<Apple> result = new Arraylist<>(); // 사과 리스트
  if (GREEN.equals(apple,getColor()) { // 녹색 사과 선택
   result.add(apple);
  }
 }
return result;
}
```
위의 코드는 녹색사과만 필터링 하고 있다. 빨간 사과로 필터링 하고 싶다면 메서드를 복사해서 filterRedApples라는 새로운 메서드를 만들고,    
if문의 조건을 빨간 사과로 바꾸는 방법을 선택할 수 있다. 하지만 반복되는 코드가 존재한다면 추상화 할 수 있다.


### 2. 색을 파라미터화 (추상화)
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
 List<Apple> result = new Arraylist<>() ;
   for (Apple apple: inventory) {
    if (apple.getColorO .equals(color)) {
     result.add(apple);
    }
   }
  return result;
}
```
filterRedApples라는 코드를 반복 사용하지 않고 색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하여 유연하게 대응하는 코드를 만들 수 있다.     
하지만 목록을 검색하고, 각사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복된다.  
이는 소프트웨어 공학의 DRY(don't repeat yourself : 같은 것을 반복하지 말 것) 원칙을 어기는 것이다.  
성능을 개선하려면 메서드 전체를 고쳐야하기 때문에 엔지니어링적으로 비싼 대기를 치러야 한다.    
그렇기 때문에 위의 코드는 동작 파라미터화를 통해 개선할 수 있다.

### 3.동작 파라미터화
boolean을 반환하는 Predicate(선택 조건을 결정하는 인터페이스)를 정의한다.
```java
public class ApplePredicate {
   boolean test(Apple apple);
 }
```

```java
  public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }
  }

  public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }
  }
```
위의 코드처럼 다양한 ApplePredicate를 정의할 수 있다.
이를 전략 디자인 패턴 (strategy design pattern)이라고 부르며,    
각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법이다.    
위의 코드에선 ApplePredicate가 알고리즘 패밀리고 AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략이다.


### 4. 추상적 조건으로 필터링
filterApples 메서드가 ApplePredicate를 객체를 인수로 받도록 수정하면,   
filterApples 메서드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작(Predicate)을 분리할 수 있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득을 얻는다.
```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
 List<Apple> result = new ArrayList<>();
  for (Apple apple : inventory) {
   if (p.test(apple)) {
    result.add(apple);
   }
  }
  return result;
}
```

### 5. 익명 클래스 사용
익명클래스는 말 그대로 이름이 없는 클래스다.   
익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다. 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다.
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
 public boolean test(Apple a) {
  return RED.equals(apple.getColor());
 }
});
```

### 6. 람다식 사용
람다 표현식을 이용해 한눈에 이해할 수 있는 코드로 간결하게 정리할 수 있다.
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```   
<br>

### 리스트 형식으로 추상화
파라미터 T를 이용하면 사과 이외에 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용하여 활용할 수 있다.
```java
public interface predicate<T> {
  boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for(T e : list) {
    if(p.test(e)) {
      result.add(e);
    }
  }
  return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Apple> evenNumers = filter(numbers, (Integer i) -> i % 2 == 0);
```
<br>

> [참고하면 좋을 내용들]
> - 람다가 인스턴스화 되는 jvm 내부 구조 찾아보기
> - 이펙티브 자바
    >   - 아이템 42. 익명 클래스 보다는 람다를 사용하라
>   - 아이템 43. 람다보다는 메서드 참조를 사용하라
>   - 아이템 44. 표준 함수형 인타페이스를 사용하라
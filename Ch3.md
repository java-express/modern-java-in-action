# 모던 자바인 액션 ch3 - 람다 표현식

### 람다란 무엇인가?

**람다 표현식**은 메서드로 전달할 수 있는 <u>익명 함수를 단순화한 것</u> 이라고 할 수 있다. 람다 표현식에는 이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다.
![](https://velog.velcdn.com/images/kimjinwook/post/b5812def-e30d-49d0-92c8-b990e241a44e/image.png)


### 함수 디스크립터

- 메서드 시그니처 : 메서드이름과 매개변수 리스트의 조합

- 람다의 시그니처 : 함수형 인터페이스의 추상 메서드 시그니처

- 함수 디스크립터 : 람다의 시그니처를 서술하는 메서드


### @FunctionalInterface

: 함수형 인터페이스임을 가리키는 어노테이션

추상 메서드가 **두 개 이상**이라면 컴파일 에러가 발생한다.

- java.lang.Object의 공용 메서드 중 하나를 재정의 하는 추상메서드를 선언하는 것은 추상 메서드의 수에 포함되지 않는다.

https://mkyong.com/java8/is-comparator-a-function-interface-but-it-has-two-abstract-methods/

### 람다 활용

람다와 동작 파라미터화로 유연하고 간결한 코드를 구현하는데 도움을 주는 실용적인 예제를 살펴보자.

```java
public String processFile() throws IOException {  
     try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {  
         return br.readLine();  
     }  
}
```

위와 같은 코드를 람다를 활용하여 변경시켜보자.

- 1단계 : 동작 파라미터화를 기억하라


```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

동작 파라미터화를 통해 코드를 최종적으로 위와 같이 변경시켜야 한다.

- 2단계 : 함수형 인터페이스를 이용해서 동작 전달


```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}
```

위와 같이 정의한 함수형 인터페이스를 통해 processFile 메서드의 인수로 전달할 수 있다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```

- 3단계 : 동작 실행


```java
private String processFile(BufferedReaderProcessor p) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return p.process(br);
	}
}
```

함수형 인터페이스를 processFile 메서드의 인수로 전달하였고, 추상 메서드 구현을 직접 전달할 수 있으며 전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리한다.

- 4단계 : 람다 전달


이제 람다를 이용하여 다양한 동작을 processFile 메서드로 전달할 수 있다.

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());

Stirng twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());

```

위의 람다 형식을 익명함수로 풀어서 사용하면 다음과 같다.

```java
String oneLine = processFile(new BufferedReaderProcessor() {
	@Override
	public String process(BufferedReader br) throws IOException {
	    return br.readLine();
	}
});


String twoLine = processFile(new BufferedReaderProcessor() {
	@Override
	public String process(BufferedReader br) throws IOException {
	    return br.readLine() + br.readLine();
	}
});
```

함수형 인터페이스를 직접 정의하고 람다를 활용하여 코드를 더 간결하게 사용할 수 있게 된다.

### 함수형 인터페이스

자바 8 라이브러리 설계자들은 java.util.function 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.

**Predicate**

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

**Consumer**

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

**Function**

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

###

### 형식 추론

```java
Predicate<String> predicate = (String s1) -> s1.isEmpty();

Predicate<String> predicate = (s1) -> s1.isEmpty();
```

파라미터의 형식을 배제 시켜 코드를 더 깔끔하게 사용 가능하다. 하지만 형식을 배제시키지 않는다고 하더라도 나쁜 코드는 아니다. 형식을 한 눈에 알 수 있으므로 가독성이 더 좋을 수 도 있다.

### 지역 변수 사용

**람다 캡쳐링** -> 람다에서 사용한 지역변수를 한번 더 값을 재할당 하는 경우 컴파일할 수 없다.

```java
int number = 0;
Runnable r = () -> System.out.print(number); // -> number에서 컴파일에러가 발생한다.
number = 1; // 이미 할당된 지역변수의 값을 한번 더 재할당 하였으므로 람다 캡쳐링이 발생한다.
```

> 지역 변수의 제약이 생기는 이유는 인스턴스 변수는 힙에 저장되지만 지역 변수는 스택에 위치한다. 람다가 지역 변수에 바로 접근 가능하다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다. 따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 **지역 변수의 복사본**을 제공한다. 따라서 **복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴 것**이다.

### 메서드 참조

1. 정적 메서드 참조 -> **Integer::parseInt**

2. 다양한 형식의 인스턴스 메서드 참조 -> **String::length**

3. 기존 객체의 인스턴스 메서드 참조
   예를 들어 Transaction 객체를 할당받은 expensiveTransaction 지역 변수가 있고, Transaction 객체에는 getValue 메서드가 있다면, **expensiveTransaction::getValue** 라고 표현할 수 있다.


| 람다  | 메서드 참조 |
| --- | --- |
| (Apple apple) -> apple.getWeight() | Apple::getWeight |
| () -> Thread.currentThread().dumpStack() | Thread.currentThread()::dumpStack |
| (str, i) -> str.substring(i) | String::subString |
| (String s) -> System.out.println(s) | System.out::println |
| (String s) -> this.isValidName(s) | this::isValidName |

### 생성자 참조

클래스명과 new 키워드를 이용하여 기존 생성자의 참조를 만들 수 있다.

```java
Supllier<Apple> c1 = Apple::new;
Apple a1 = c1.get(); // <- 새로운 Apple 객체가 생성된다.
```

```java
Function<Integer, Apple> c2 = Apple::new;
Apple a2 = c2.apply(110); // <- Integer를 인수로 받는 Apple 객체가 생성된다.
```

**비공개 헬퍼 메서드**
: 어떤 클래스의 메서드에서 해당 클래스의 private 메서드를 사용하는 것을 비공개 헬퍼 메서드를 사용한다고 한다.

```java
private boolean isValidName(String string) {
    return Character.isUpperCase(string.charAt(0));
}

filter(words, this::isValidName)
```

###

### 람다, 메서드 참조 활용하기

- 1단계. 코드전달

  ```java
  void sort(Comparator<? super E> c)
  ```

  이 코드는 Comparator 객체를 인수로 받아 객체를 비교한다. sort의 동작은 파라미터화 되었다.

  ```java
  public class AppleComparator implements Comparator<Apple> {
  	public int compare(Apple a1, Apple a2) {
  		return a1.getWeight().compareTo(a2.getWeight());
  	}
  	inventory.sort(new AppleComparator());
  } 
  ```

- 2단계. 익명 클래스 사용

  ```java
  inventory.sort(new Comparator<Apple>) {
      public int compare(Apple a1, Apple a2){
          return a1.getWeight().compareTo(a2.getWeight());
      }
  }
  ```

- 3단계. 람다 표현식 사용

  ```java
  inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
  
  
  inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
  ```

  Comparator의 정적메서드 comparing을 사용하게되면 코드를 더 줄일 수 있다.

  ```java
  inventory.sort(comparing(apple -> apple.getWeihgt());
  ```


- 4단계. 메서드 참조

  ```java
  inventory.sort(comparing(Apple::getWeight));
  ```

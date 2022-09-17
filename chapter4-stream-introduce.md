# 1. What is Stream?

4장에서는 스트림의 개념과 스트림이 갖고 있는 중요한 특성을 설명한다.

자바 8에 새로 추가된 기능인 스트림의 핵심 패러다임은 선언형 프로그래밍에 있다. 스트림을 한마디로 정의하자면, 데이터 처리 연산을 목적으로 데이터 소스에서 추출한 연속된 요소(Sequenceof elements)라고 할 수 있다. 스트림의 정의속에서 스트림이 가진 중요한 특성을 파악할 수 있다.

> **1. 데이터 처리 연산을 목적으로**
> **2. 데이터 소스에서 추출한**
> **3. 연속된 요소(Sequenceof elements)**



## 1-1. 데이터 처리 연산을 목적으로 만들어졌다.


스트림이 탄생한 이유는 무엇일까? 단순하게 말하자면, 스트림은 데이터 컬렉션 반복을 멋지게 처리하기 위해 만들어졌다. 데이터 컬렉션을 처리하는데 필요한 메서드 대부분을 추상화하고 있기 때문에 명시적으로 코드를 작성할 수 있다. 스트림은 컬렉션, 배열, I/O 자원 등에서 데이터 소스를 제공받아 연산 처리하는데 특화되어 있다. 스트림을 생성하면 데이터 소스와 동일한 데이터 순서가 보장되기 때문에 마치 데이터 소스에 직접 접근하여 연산을 처리하는 것처럼 보일 수 있다. 하지만 대표적인 데이터 소스인 컬렉션과 비교해보면 스트림의 관심사가 조금 다른 것을 알 수 있다. 컬렉션은 자료구조이미로, 데이터의 저장과 접근에 필요한 연산이 주된 관심사다. 반면, 스트림은 filter, sorted, map처럼 표현 계산식이 핵심 기능을 이룬다. 즉, 컬렉션의 관심사는 데이터에 있고, 스트림의 관심사는 계산(연산)에 있다.


## 1-2. 데이터 소스에서 추출한 연속된 요소를 다룬다

앞에서도 언급했지만, 스트림의 데이터 순서는 오리지널 데이터 소스의 순서와 동일하다. 스트림은 데이터 소스를 기준으로 구조체를 생성하기 때문에 정렬된 컬렉션으로 스트림을 생성하면 동일한 데이터 정렬이 스트림 내에서도 그대로 유지된다.  따라서 데이터 컬렉션을 직접 다루듯 동일한 값이 모인 인터페이스를 선언적으로 조작할 수 있다.


위와 같은 스트림의 특징은 스트림만의 강력한 장점이기도 하다. 스트림은 어떻게 특징을 강점으로 만들 수 있었을까? 바로 데이터 처리 연산을 원하는 방식으로 조립할 수 있도록 도와주는 파이프라닝 기능과 내부 코드를 몰라도 데이터 연산이 가능한 내부 반복 기능이다.



# 2. 스트림을 사용하면 무엇이 좋은가?

파이프라이닝과 내부 반복은 스트림을 활용하는 가장 중요한 키워드가 된다. 스트림이 가진 두 가지 기능은 선언형 프로그래밍 패러다임이 적용되면서 가능해졌다. 선언형 코드 구현이 가능하다는 점은 스트림이 가진 확실한 소프트웨어 공학적 이점이다. 특히, 동작 파라미터와 함께 활용하면 변화하는 요구사항에 맞춰 쉽게 코드를 수정하는 것이 가능하다. 그리고 filter, sorted, map, collect 와 같은 메서드가 어떤 기능을 하는지 정도만 이해하고 있다면, 여러가지 메서드 블럭을 조립하여 연속된 데이터를 원하는 방식으로 가공할 수 있다. 높은 수준의 추상화를 제공하는 선언형 프로그래밍으로 인해 복잡한 연산이 합쳐진 파이프라이닝을 구현해도 가독성과 명확성을 해치지 않는다. 아울러, 내부 구현을 직접 신경쓰지 않아도 되기 때문에 병렬 작업이 필요할 때도 스레드나 락에 대해 크게 고민할 게 없다.


# 3. 스트림의 동작 흐름


스트림이 제공하는 높은 수준의 추상화를 통해 간단하게 데이터 컬렉션을 가공할 수 있다는 것을 알게 되었다. 그렇다면 어떻게 활용할 수 있을까? 먼저 스트림의 동작 방식을 이해해야 한다. 스트림의 동작 흐름은  크게 3단계로 구분된다.

> 스트림의 동작 흐름: 스트림 생성 -> 중간 연산 -> 최종 연산(결과 만들기)
>![](https://velog.velcdn.com/images/woply/post/09dc9a3b-bfc2-4488-a830-80abe2542869/image.png)


## 3-1. 스트림 생성

스트림 인스턴스를 생성하기 위해서는 데이터 소스가 필요하다. 일반적으로 배열이나 컬렉션 인스턴스를 이용해 생성하지만, 이 외에도 다양한 방식으로 스트림을 만들 수 있다. 몇 가지 스트림 생성 방식을 살펴보자.

배열 스트림은 Arrays.stream()을 사용하여 스트림 생성이 가능하다.

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // *1-2 요소 [b, c]*
```

Collection, List, Set, Queue과 같은 컬렉션 타입은 인터페이스에 추가된 디폴트 메서드 stream()을 이용해 스트림 생성이 가능하다.

```java
public interface Collection<E> extends iterable<E> {
    default Stream<E> stream() {
        return StreamSupport.stream(spliteratore(), false);
    }
    // . . .
}
```

```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // *병렬 처리 스트림*
```

Stream.empty()를 이용해 데이터 소스가 없는 빈 스트림 생성도 가능하다.

```java
public Stream<String> streamOf(List<String> list) {
    return list == null || list.isEmpty() ? Stream.empty() : list.stream();
}
```

Stream.builder()를 사용하면 스트림에 직접 원하는 데이터를 넣을 수 있다.

```java
Stream<String> builderStream =
    Stream.<String>builder()
        .add("Eric").add("Elena").add("Java")
        .build(); // *[Eric, Elena, Java]*
```

Stream.generate() 메소드를 이용하면 Supplier<T> 자리에 람다를 인자로 전달할 수 있다. Supplier<T>가지고 있는 get()는 인자를 받지 않고, 리턴 값만 있는 함수형 인터페이스다. 람다식으로 익명 객체를 만들어 전달할 수 있다.

```java
public static<T> Stream<T> generate(Supplier<T> s) { . . .}
```

generate()를 사용할 때는 스트림의 크기를 지정해줘야 한다.

```java
Stream<String> generatedStream = 
    Stream.generate(() -> "gen").limit(5); // *[ gen, gen, gen, gen, gen ]*
```

그 외에 기본형 타입 스트림, 문자열 스트림, 파일 스트림 등 다양한 스트림 생성이 가능하다. 마지막으로 Stream.concat()를 이용하면 두 개의 스트림을 연결하여 새로운 스트림을 만들어낼 수 있다.

```java
Stream<String> stream1 = Stream.of("Java", "Modern", "Action");
Stream<String> stream2 = Stream.of("Jaemin", "Jinwook", "chanwoo");
Stream<String> concat = Stream.concat(stream1, stream2);
// [Java, Modern, Action, Jaemin, Jinwook, chanwoo]
```




## 3-2. 스트림 중간 연산(=가공)


스트림은 고수준의 빌딩 블록 방식으로 중간 연산을 지원한다. 높은 수준의 추상화를 통해서 사용자 친화적인 인터페이스 조작만으로 데이터 연산(=가공)이 가능하다. 또한 스트림의 중간 연산 메서드는 모두 리턴 타입이 Stream이므로, 또 다른 스트림 중간 연산 작업의 대상이 된다. 즉, 앞 뒤가 똑같은 기차의 객실처럼 필요한 연산을 원하는 방식으로 조립하여 사용할 수 있다. 이와 같은 작업을 파이프라이닝(작업 이어 붙이기)이라고 한다.

![](https://velog.velcdn.com/images/woply/post/f9a52907-c6c9-4ffe-ac3a-04a8dd06e2d8/image.png)


파이프라이닝이 가능한 대표적인 스트림 중간 연산 메서드를 살펴보자.

Filter()는 스트림 요소들을 하나씩 평가한다. 인자로 받는 Predicate는 Boolean을 리턴하는 test()를 가진 함수형 인터페이스다.

```java
Stream<T> filter(Predicate<? supter T> predicate);
```

filter()를 사용하면 스트림 데이터에 대해 평가식을 실행한다. 람다식으로 생성한 익명 객체를 통해 "a"란 문자를 포함한 인스턴스가 있을 경우만 모아서 Stream을 리턴한다.

```java
List<String> names = Arrays.asList("Modern", "Java", "Action");

Stream<String> stream =
    names.stream()
    .filter(name -> name.contains("a")); // [Java, Action]
```

Map()은 스트림의 데이터 요소들을 특정한 값으로 변환할 때 사용한다. 스트림에 들어가 있는 요소들이 순차적으로 변환 로직을 거친 후 새로운 Stream에 담긴다. 값을 변환하는 로직은 람다식으로 받을 수 있다. 이와 같은 작업을 매핑(Mapping)이라고 한다. 매핑은 다양한 방식으로 활용할 수 있다.

```java
<R> Stream<R> map(Fucntion<? super T, ? extends R> mapper);
```

아래 예제는  toUpperCase()를 이용해  스트림 내 String 인스턴스를 모두 대문자로 변환하여 또 다른 Stream을 생성하는 예제다.

```java
List<String> names = Arrays.asList("Modern", "Java", "Action");

Stream<String> stream =
    names.stream()
    .map(String::toUpperCase); // [MODERN, JAVA, ACTION]
```

map()을 사용하면 데이터를 변환하여 새로운 스트림을 만드는 방식 외에도 인스턴스가 가진 데이터를 추출하는 방식으로 Stream을 생성할 수도 있다. 아래 예제는 Person 인스턴스가 가지고 있는 나이 정보를 꺼내오는 예제다. 변환이라는 관점에서 Person 스트림을 Person의 Age로 매핑하는 것이다.

```java
Stream<Integer> stream =
    personList.stream()
    .map(Person::getAge); // [33, 30, 25]
```

중첩된 구조를 한 단계 제거하고 단일 컬렉션으로 만들어주는 flatMap도 활용도가 높다. flatMap은 인자로 mapper를 받고, 리턴타입이 Stream이다.
즉 새로운 스트림을 생성해서 리턴하는 람다식을 인자로 넘겨야 한다. 이러한 작업을 플래트닝이라고 한다.

아래는 중첩된 리스트 구조를 한번에 제거하는 flatMap 예제다.
```java
List<List<String>> list = 
    Arrays.asList(Arrays.asList("a"), Arrays.asList("b")); // *[[a], [b]]*

List<String> flatList = 
    list.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // [a, b]
```

보다 실용적인 예제를 살펴보자. 아래 예제는 students 스트림에서 학생들의 과목 점수를 받아 새로운 스트림을 생성하고, 평균을 구하는 예제다. flatMap()을 이용하면, map()으로 한 번에 처리할 수 없는 이중 스트림 구조를 처리할 수 있다.
```java
students.stream()
    .flatMapToInt(student -> IntStream.of(student.getKor(),
                                          student.getEng(),
                                          student.getMath()))
    .average().ifPresent(avg -> System.out.println(Math.round(avg * 10) / 10.0));

```


sorted()는 다른 정렬 메서드와 마찬가지로 Comparator를 사용한다. 메서드 시그니처가 두 종류인데,  인자가 없는 sorted()를 사용할 경우 기본적으로 오름차순으로 정렬한다. 인자를 넘기는 sorted()를 사용하면 원하는 조건으로 정렬할 수 있다.

```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
```

```java
IntStream.of(14, 11, 20, 39, 23)
    .sorted() // 인자를 사용하지 않으면 기본적으로 오름차순 적용
    .boxed()
    .collect(Collectors.toList()); // *[11, 14, 20, 23, 39]*
```

```java
List<String> lang =
    Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

lang.stream()
    .sorted()
    .collect(Collectors.toList()); // [Go, Groovy, Java, Python, Scala, Swift]

lang.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList()); // [Swift, Scala, Python, Java, Groovy, Go]
```



## 3-3. 최종 연산(결과 만들기)



중간 연산을 통해 가공한 스트림을 가지고 최종적으로 결과를 만드는 단계라고 할 수 있다. 스트림을 마무리하는 최종 작업이라는 의미에서 터미널 연산(Terminal Operation)이라고도 부른다.

스트림은 다양한 계산(Calulating) API를 제공한다. 최대, 최소, 평균, 합계 메서드 등을 이용해 기본적인 결과를 만들 수 있다. 아래는 카운트와 합계를 최종 연산하는 메서드다. 만약 스트림이 비어 있다면, count 와 sum 은 0을 출력한다.


```java
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum= LongStream.of(1, 3, 5, 7, 9).sum();
```

최소값과 최대값은 반환 타입이 Optional이다. 스트림이 비어있을 경우 표현이 불가하기 때문이다.
```java
OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();
```

스트림에서 바로 ifPresent 메소드를 활용해 Optional을 처리할 수 있다.
```java
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5)
    .average()
    .ifPresent(System.out::println);
```

스트림은 reduce()를 이용해 결과를 만들 수 있다. 스트림에 있는 여러 요소의 총합을 연산하는 것도 가능하다. reduce()는 3가지 파라미터 타입을 가진다.

accumulator : 각 요소를 처리하는 계산 로직이다. 각 요소가 올 때마다 중간 결과를 생성한다.
identity : 계산을 위한 초기값으로 스트림이 비어있어 계산할 내용이 없더라도 이 값이 리턴된다.
combiner : 병럴(parallel) 스트림에서 나눠 계산한 결과를 하나로 합쳐주는 로직이다

```java
Optional<T> reduce(BinaryOperator<T> accumulator); // 1개의 파라미터를 받는 reduce()
T reduce(T identity, BinaryOperator<T> accumulator); // 2개의 파라미터를 받는 reduce()
<U> U reduce(U identity, <U, ? super T, U> accumulator, <U> combiner); // 3개의 파라미터를 받는 reduce()

/*
accumulator : 각 요소를 처리하는 계산 로직이다. 각 요소가 올 때마다 중간 결과를 생성한다.
identity : 계산을 위한 초기값이다. 스트림이 비어있어 계산할 내용이 없을 경우 이 값이 리턴된다.
combiner : 병럴(parallel) 스트림에서 나눠 계산한 결과를 하나로 합쳐주는 로직이다.
*/
```

인자가 하나만 있는 reduce()부터 살펴보자. BinaryOperator<T> 는 같은 타입의 인자(T) 두 개를 받아 같은 타입의 결과를 반환하는 함수형 인터페이스다. 아래 예제는 두 값을 더 하는 람다식을 메서드의 구현 로직으로 전달한다.
```java
// Optional<T> reduce(BinaryOperator<T> accumulator); // 1개의 파라미터를 받는 reduce()

OptionalInt reduced = 
    IntStream.range(1, 4) // *[1, 2, 3]*
    .reduce((a, b) -> Integer.sum(a, b)); // 결과 6
```

이번엔 두 개의 인자를 받는 reduce다. 여기서 10은 초기값이고, 스트림 내 값을 더하여 결과는 16(10 + 1 + 2 + 3)이 된다. 여기서 람다는 메소드 참조(method reference)를 이용하여 넘길 수 있다.

```java
// T reduce(T identity, BinaryOperator<T> accumulator); // 2개의 파라미터를 받는 reduce()

int reducedTwoParams =
    IntStream.range(1, 4) // *[1, 2, 3]
    .reduce(10, Integer::sum);* // *method reference*
```

마지막으로 세 개의 인자를 reduce()다. Combiner가 하는 역할을 코드로 한 번 살펴보자. 아래의 코드를 실행할 경우 마지막 인자인 Combiner는 실행되지 않는다. 병렬 스트림이 아니기 때문이다.
```java
// <U> U reduce(U identity, <U, ? super T, U> accumulator, <U> combiner); // 3개의 파라미터를 받는 reduce()

Integer reducedParams = Stream.of(1, 2, 3)
    .reduce(10, Integer::sum, (a, b) -> {
                        System.out.println("Combiner was called");
                        return a + b;
                    });
```

Combiner는 병렬 처리 시 각자 다른 쓰레드에서 실행한 결과를 마지막에 합치는 단계이다. 따라서 병렬 스트림에서만 작동한다. 아래 코드를 실행하면 결과는 36이 나온다. 스트림은 lazy하게 동작하는 특징을 이해해야 한다. 먼저 accumulator가 총 세 번 동작한다는 점을 기억하자. 초기값 10에 각 스트림 값을 개별적으로 합산하는 연산을 수행한다. 10 + 1 = 11,  10 + 2 = 12, 10 + 3 = 13을 병렬로 계산한다. Combiner는 identity와 accumulator를 가지고 여러 쓰레드에서 나눠 계산한 결과를 합치는 역할을 한다. 앞서 3번의 accumulator동작을 통해 얻은 11, 12, 13을 각각 12 + 13 = 25, 25 + 11 = 36 이렇게 두 번의 호출과 실행을 거친다. 이처럼 간단한 연산의 경우 병렬 작업은 부가적인 처리를 필요로하기 때문에 비효율적일 수 있다.

```java
Integer reducedParallel = Arrays.asList(1, 2, 3)
    .parallelStream()
    .reduce(10, Integer::sum, (a, b) -> {
                        System.out.println("Combiner was called");
                        return a + b;
                    });
```

collect() 또 다른 종료 작업이다. Collector 타입의 인자를 받아서 처리 하는데, 자주 사용되는 작업은 Collectors 객체가 제공한다. 아래 예제에선 다음과 같은 간단한 리스트를 사용한다. Product 객체는 수량(amount)과 이름(name)을 가지고 있다.
```java
List<Product> productList =
    Arrays.asList(new Product(23, "potato"),
                                new Product(14, "orange"),
                                new Product(13, "lemon"),
                                new Product(23, "bread"),
                                new Product(13, "sugar"),
```
Collectors.toList()은 스트림에서 작업한 결과물을 List로 반환한다. 아래 예제는 map() 으로 각 요소의 이름을 가져온 후 Collectors.toList 를 이용해 리스트로 결과를 가져온다.
```java
List<String> collectorCollection =
    productList.stream()
        .map(Product::getName)
        .collect(Collectors.toList()); // [potato, orange, lemon, bread, sugar]
```

Collectors.joining()을 사용하면 스트림에서 작업한 결과를 하나의 스트링으로 이어 붙일 수 있다. Collectors.joining()은 세 개의 인자를 받을 수 있다. 이를 이용하면 간단하게 스트링을 조합할 수 있다.

```java
/*
delimiter : 각 요소 중간에 들어가 요소를 구분 시켜주는 구분자
prefix : 결과 맨 앞에 붙는 문자
suffix : 결과 맨 뒤에 붙는 문자
*/

String listToString =
    productList.stream()
        .map(Product::getName)
        .collect(Collectors.joining()); // potatoorangelemonbreadsugar

String listToString = 
    productList.stream()
        .map(Product::getName)
        .collect(Collectors.joining(", ", "<", ">")); // <potato, orange, lemon, bread, sugar>
```

Collectos.groupingBy()를 사용하면 특정 조건으로 요소를 그룹핑할 수 있다. 수량을 기준으로 그룹핑해보자. 여기서 받는 인자는 함수형 인터페이스 Function이다. 결과는 Map 타입으로 반환되며, 같은 수량이면 리스트로 묶어 나타난다.

```java
Map<Integer, List<Product>> collectorMapOfLists =
    productList.stream()
        collect(Collectors.groupingBy(Prodcut::getAmount));

/*
결과
{23=[Product{amount=23, name='potato'}, 
     Product{amount=23, name='bread'}], 
 13=[Product{amount=13, name='lemon'}, 
     Product{amount=13, name='sugar'}], 
 14=[Product{amount=14, name='orange'}]}
*/
```

매칭(Matching)은 조건식 람다 predicate를 받아 해당 조건을 만족하는 요소가 있는 지 체크한 결과를 반환한다. 매칭은 세 가지 메소드를 제공한다.

```java
/*
anyMatch(): 하나라도 조건을 만족하는 요소가 있는지 검사
allMatch(): 모두 조건을 만족하는지 검사
noneMatch(): 모두 조건을 만족하지 않는지 검사
*/

boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
```

아래의 결과는 모두 true를 반환한다.

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
  .anyMatch(name -> name.contains("a"));
boolean allMatch = names.stream()
  .allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream()
  .noneMatch(name -> name.endsWith("s"));
```


foreach()는 요소를 돌면서 실행되는 최종 작업이다. 보통 Systrem.out.println 메소드를 넘겨서 결과를 출력할 때 사용한다. 중간 연산에서는 peek()가 비슷한 기능을 한다. 중간 연산이 아니기 때문에 Stream을 반환하지는 않는다.
```java
names.stream().forEach(System.out::println);
```



# 4. 스트림과 컬렉션의 차이점


스트림과 컬렉션은 모두 연속된 요소 형식의 값을 저장하는 자료구조형 인터페이스를 제공한다. 순차적으로 값을 제공한다는 공통점이 있다. 그렇다면 차이점은 무엇일까? 크게 3가지 차이점이 있다.

> 1. 데이터 계산 시점
> 2. 반복의 일회성
> 3. 외부반복과 내부반복




## 4-1. 데이터 계산 시점

데이터의 계산 시점이 스트림과 컬렉션의 가장 큰 차이라고 할 수 있다. 컬렉션은 모든 요소를 컬렉션에 추가하기 전에 계산한다. 반면 스트림은 요청 시점에 계산한다. 마치 게으르게 만들어지는 컬렉션과 같다. 스트림은 사용자의 요청이 있을 때, 요청한 값만 추출한다는 점에서 프로그래밍에 장점이 있다.


## 4-2. 반복의 일회성

컬렉션과 스트림은 반복처리 방식의 차이가 있다. 컬렉션은 데이터 소스에 대해 여러번 반복 처리가 가능하다. 하지만 스트림은 딱 한 번만 처리할 수 있다. 스트림은 소비의 개념에 가깝다. 한 번 소비한 데이터 소스는 다시 접근할 수 없다. 만약 아래 코드를 실행한다면 stream has already been operated upon or closed 라는 에러와 함께 프로그램이 종료된다.

```java
Stream<Food> s = foodList.stream();
s.forEach(System.out::println); // 정상
s.forEach(System.out::println); // IllegalStateException 발생

```



## 4-3. 외부반복과 내부반복의 차이


컬렉션은 foreach 문법을 사용하여 사용자가 반복문을 직접 처리해야 한다. 이와 같은 방식을 외부반복이라고 한다. 반면 스트림은 반복을 처리하는 로직이 라이브러리 안에 감춰져 있다. 실제 로직이 내부에 있기 때문에 내부 반복이라고 한다. 두 가지 방식의 Food 리스트의 이름을 추출 코드를 통해 직접 비교해보자. 컬렉션은 반복문을 개발자가 직접 구현한다. 반면 스트림은 반복의 결과를 선언적으로 명령한다. 스트림은 내부 반복 방식을 통해 작업을 병렬로 처리하거나, 더 효율적인 반복 로직을 수행할 수 있다.

[컬렉션]

```java
List<String> foodNameList = new ArrayList<>();
for(Food food : foodList){
    foodNameList.add(food.getName());
}
```


[스트림]

```java
List<String> foodNameList = foodList.stream()
        .map(Food::getName)
        .collect(Collectors.toList());

```




# 5. 정리하기


스트림의 개념과 특징을 정리해보자. 스트림은 데이터 소스에서 추출된 연속된 요소라고 할 수 있다. 스트림의 근본적인 목적은 데이터 처리 연산이다. 스트림의 고수준의 추상화를 통해 선언형 프로그래밍을 가능하게 한다. 반복에 필요한 로직은 내부 반복 방식으로 동작한다. 내부 반복은 중간 연산에 사용되는 메서드에 사용된다. 스트림의 동작 흐름은 스트림 생성 - 중간 연산 - 결과 만들기의 과정으로 이어진다. 중간 연산에 해당하는 메서드는 동일한 리턴타입을 제공하기 때문에 원하는 방식으로 연결하여 사용할 수 있다. 이와 같은 방식을 파이프라이닝이라고 한다. 중간 연산만으로는 결과를 만들 수 없기 때문에 스트림의 마지막 단계에서는 최종 연산(결과 만들기)에 해당하는 메서드가 필요하다. 대표적인 메서드로는 forEach나 count가 있다. 최종 연산 메서드는 리턴 타입이 Stream이 아닌 결과를 반환한다. 마지막으로 스트림의 중요한 특징 중 하나는 한 번에 하나씩 그리고 게으르게 계산한다. 이는 요청이 발생하는 시점에 해당하는 요소만 연산을 처리하는 것을 의미한다.
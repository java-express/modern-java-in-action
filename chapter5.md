## 스트림 활용 

#### | 필터링, 슬라이싱, 매칭
필터링은 `데이터 컬렉션 반복을 내부적`으로 처리한다. 
`내부 반복`을 이용하게 되면 Stream API 가 데이터 관련 작업을 관리하므로 
1) 내부적으로 다양한 최적화 가능하며
2) 코드를 병렬 실행 여부도 결정 가능하다. 

> 요소 필터링 방법 

필터의 방법에서는 1) Predicate필터링 방식과 2) 고유 요소 필터링이 존재한다. 

filter() 메서드는 `Predicate`(returns boolean) 를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다. 

`고유 요소` 로 이루어진 스트림을 반환하는 `distinct` 메소도 지원하며 객체의 hashCode 와 equals 로 결정된다. 

> 스트림 슬라이싱 

스트림의 요소를 선택하거나 스킵하는 방법으로는 하기 방법들이 있다. 
1) 프레디케이트 이용 방법
2) 스트림의 처음 몇개의 요소를 무시하는 방법
3) 특정 크기로 스트림을 줄이는 방법

[1] Predicate 이용 방법   

Java9 부터는 `takeWhile` 과 `dropWhile` 을 이용하여 효과적으로 요소 선택이 가능하다. 

`takeWhile` 이용하면 무한 스트림을 포함한 모든 스트림에 Predicate 적용해 스트림을 슬라이스 할 수 있다. 동작 방법이 얼핏보면 filter 와 비슷하지만 filter 는 전체 스트림에 대한 Predicate 를 판단하는 반면
`takeWhile`은 Predicate 가 false 를 반환하는 순간 나머지를 버린다. 그렇기 때문에 <u>무한 스트림에서도 적용하능하며 정렬된 요소에서 가장 큰 장점을 가질수 있다.</u>

<img width="570" alt="Screen Shot 2022-09-30 at 11 38 37 PM" src="https://user-images.githubusercontent.com/16564373/193294368-7b9dfb26-841a-4076-b1f7-d37c4e26069c.png">

`dropWhile` 은 `takeWhile` 정반대의 동작으로 Predicate 가 거짓이 되는 지점에서 작업을 종료하고 나머지 요소들을 반환한다. `dropWhile` 역시 무한 스트림에서도 동작한다. 

<img width="544" alt="Screen Shot 2022-09-30 at 11 39 22 PM" src="https://user-images.githubusercontent.com/16564373/193294499-8eb87d31-7139-46ac-b9f4-776217c88724.png">

> 스트림 축소 

`limit(n)` : 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환할 수 있게 해준다. 
`skip(n)` : 처음 n 개의 요소를 제외한 스트림 반환 

limit 과 skip 은 상호보완적인 연산을 수행한다. 
상호 보완적인 연산이라는 것은 limit을 통해 순서대로의 n 개의 크기를 반환하고 나머지는 skip 으로 추출이 가능하다.   

<img width="562" alt="Screen Shot 2022-09-30 at 11 39 45 PM" src="https://user-images.githubusercontent.com/16564373/193294587-a2f2b85a-f55b-4b1a-b429-fedf6d1cb95a.png">


#### | 검색, 매칭, 리듀싱
> 매칭 

특정 데이터를 선택하는 작업은 `map` 그리고 `flatMap` 기능을 제공한다. 

* `map(String::length)` : 인수로 제공된 함수는 각 요소에 적용되며 적용한 결과가 새로운 요소로 매핑된다. 
* `flatMap()` : 고유 문자로 이루어진 리스트 반환은 `distinct()`를 사용하는 방법이 아닌 해당 함수를 사용해야 한다. 

여기서 고유 문자로 만들기 위해서는 배열 스트림 대신 문자열 스트림이 필요하며 <u> 각 단어를 개별 문자열로 이루어진 배열로 만든 다음 각 배열을 별도의 슽림으로 만들어야 한다. </u> 

```java
List<String> uniqueCharacter = 
	words.stream()
		.map(word -> word.split("")) // 각 단어를 개별의 문자 포함 배열로 변환
		.flatMap(Arrays::stream)     // 생성된 스트림을 하나의 스트림을호 평면화
		.distinct()
		.collect(toList());
```

⭐  `flatMap` 은 배열의 스트림이 아닌 스트림의 콘텐츠로 매핑한다는 것이 중요하다. 
`Function<T, Stream<R>>` 을 통해 하나의 스트림으로 연결을 수행하는데 `R`은 객체가 아닌 일반 스트림이나 기본형 특화 스트림(IntStream, LongStream, DoubleStream) 같이 스트림 타입을 반환한다. 

<img width="549" alt="Screen Shot 2022-09-30 at 11 40 01 PM" src="https://user-images.githubusercontent.com/16564373/193294655-58fb7894-6561-4a6f-951b-1b71dbe9d493.png">

> 검색과 매칭 

특정 속성이 데이터 집한에 있는지 검색할때는 아래 스트림 API 가 사용된다. 
* `allMatch()` : 스트림의 모든 요소가 주어진 Predicate 와 일치하는지 검사 
* `anyMatch()` : `Predicate` 를 통해 주어진 스트림에서 적어도 한 요소와 일치하는지 확인시 사용한다. 
* `noneMatch()` : allMatch() 와 반대 연산자 수행하며, 프레디케이트와 일치하는 요소가 없는지 확인한다. 

세 메서드는 `short-circuit(쇼트 서킷)` 기법을 사용한다. 
쇼트 서킷이란 전체 스트림을 처리하지 않아도 결과를 반환할수 있으며 <u> 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이다. </u> 
그렇기에 모든 스트림을 처리하지 않고도 결과를 반환할수 있으면 요소를 찾았을때 즉시 결과를 반환할수 있기에 유한한 크기로 줄일 수 있다. `limit` 또한 해당 기법을 사용한다. 

* `findAny()` : 스트림에서 임의의 요소를 반환 한다. 

findAny() 같은 경우 스트림 파이프라인은 내부적으로 단일 과정으로 실행할 수 있도록 최적화된다. 
findAny() 를 사용할때는 return 포맷이 Optional<> 이 된다. 

`Optional` 이란? 
값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다. 
findAny() 가 아무 요소도 반환하지 않을 수 있기에 java8 부터는 Optional< T> 를 통해 null 버그를 피할수 있는 설계를 하였다. 
값이 없을 경우 강제 처리 기능을 제공하는데 `isPresent()`, `isPresent(Consumer<T> block)` `T get`, `T orElse(T other)` 이다. 

* `findFirst()` : 리스트 또는 정렬된 연속 데이터에서 (논리적인 아이템 순서가 정해져 있을 경우) 첫번째 요소를 반환한다.

여기서 신기한 점은 `findAny()`를 사용하고 `.limit(1)`을 하면 쇼티서킷 기법으로 인해 첫번쨰 요소를 반환하게 된다. 그러면 `findFirst()` 는 언제 사용해야 하는가? 
findFirst() 는 병렬 실행애서 첫번째 요소를 찾기 어렵다. 요소 반환 순서가 상관없다면 병렬 스트림에서는 `findAny` 를 사용해야 한다. 

> 리듀싱 

리듀싱 연산은 <u> 뭐든 스트림 요소를 처리해서 값으로 도출하는 것이다.</u> 함수형 프로그래밍에서는 작은 조각이 될때까지 반복해서 접는 것과 비슷하다고 하여 폴드 라고 부른다.

`.reduce()` 를 이용하면 애플리케이션의 반복된 패턴을 추상화 할 수 있따. 

예시로는 리스트의 숫자 요소를 코드를 작성해 보자. 
```java
int sum = numbers.stream().reduce(0, (a,b) -> a + b);
int sum = numbers.stream().reduce(0, Integer::sum);
```

두 요소를 조합해서 새로운 값을 만드는 BinaryOperator< T> 를 인자로 사용했다. 

```java
Optional<T> reduce(BinaryOperator<T> accumulator);

Optional<Integer> sum = numbers.stream().reduce((a, b) → (a + b));
```

초깃값을 받지 않도록 오버로드된 reduce 같은 경우는 `optional` 을 반환해야 하는데 초깃값이 없으므로 합계를 반환할수 없기에 Optional 객체로 갑싼 결과를 반환한다. 

 최대값/최소값 반환이 쉽게 가능하다.
```java
// 최댓값
Optional<Integer> max = numbers.stream().reduce(Integer::max);
// 최솟값
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```


👍  `reduce` 메서드의 장점과 병렬화?

`reduce` 를 이용하면 내부 반복이 추상화 되면서 내부 구현에서 병렬로 실횅하게 된다. 
반복적인 합계에서는 sum 변수를 공유해야해서 쉽게 병렬화가 어렵과 강제적으로 동기화 시켜도 스레드간의 소모적인 경쟁때문에 이득이 상쇄 되어 버린다. 

👍  스트림 연산: 상태 있음과 상태 없음

스트림은 원하는 모든 연산을 쉽게 구할 수 있으며 컬렉션으로 스트림을 만드는 `parallelStream` 으로 바꾸는 것만으로 병렬성을 얻을 수 있었다. 

`stateless operation`이란 내부 상태를 갖지 않는 연산자로
`map` `filter`와 같이 입력 요소를 받아 결과를 출력 스트림으로 보내는 참조가 내부적인 가변성을 갖지 아는 것을 말한다.  요소의 수와 관계없이 내부 상태의 크기는 한정으로 되어 있다. 

`stateful operation` 이란 `sorted distinct`처럼 요소를 정렬하거나 중복 제거와 같의 과거의 이력을 알고있어야 하므로 모든 <u> 요소가 버퍼에 추가되어 있어야</u> 한다. 필요한 장소는 정해져 있지 않다. 




#### | 숫자형 스트림 

스트림 API 에는 숫자 스트림을 효율적으로 처리할 수 있는 `기본 특화 스트림 (primitive stream specialization)` 을 제공한다. 

> 기본 특화 스트림

기본 특화 스트림에는 `IntStream/ DoubleStream/ LongStream` 세가지가 제공된다. 

숫자 스트림을 IntStream 으로 기본형 정수값만 반환하게 하고 boxed 메서드를 통해 특화 스트림을 일반 스트림을 반환 할 수 있다. 

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed
```

IntStream/LongStream 에서는 `range` 와 `rangeClosed` 두가지 정적 메서드를 제공한다. 

`range` 는 시작값과 종료값이 결과에 포함되지 않는 반면 
`rangeClosed` 는 시작값과 종료값이 결과에 포함된다. 

> 피타고라스의 수 

```java
.stream.filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
	.map(b -> new int[]{a, b, (int) Math.sqrt(a*a + b*v)});
```


#### | 배열로 스트림 
배열을 인수로 받는 정적 메서드 `Arrays.stream` 이용해 스트림을 만들수 있다. 

#### | 함수로 무한 스트림 생성
스트림 API 는 두 정적 메서드를 제공해 *무한스트림* 을 생성해 값을 만든다. 
따라서 무제한으로 값을 계산할 수 있으나 보통 무한한 값은 출력하지 않고 `limit` 을 사용한다. 

* Stream.iterate

`iterate` 는 요청할때마다 값을 생산할 수 있으며 끝이 없는 무한 스트림이고 이런 스트림을 `바운드 스트림`이라고 표현한다. 

iterate을 이용하여 피보나치의 수열을 찾을 수 있다. 
```java
Stream.iterate(new int[]{0, 1},
				  t-> new int[]{t[1], t[0] + t[1]})
		.limit(20)
		.map(t -> t[0])
		.forEach(System.out::println);
```


* Stream.generate

`generate` 요구할때 값을 계산하는 무한 스트림을 만들지만 iterate 와 달리 값을 연속적으로 계산하지는 않고 `Supplier< T>` 인수로 받아 새로운 값을 생산한다. 
```java
Stream.generate(Math::random)
	.limit(5)
	.forEach(System.out::println);
```

`IntSupplier` 인스턴스를 통해 피보나치 요소와 두 인스턴스 변수에 어떤 피보나치 요소가 들어가 있는지 추적하므로 가변 상태 객체인데 `getAsInt`를 호출하면 상태가 바뀌며 새로운 값을 생산한다. 

`iterate` 사용했을때 새로운 값을 생성하면서 <u>기존 상태를 바꾸지 않는 순수한 불변 상태를 유지하고 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태 기법을 고수해야한다. </u>

무한 스트림과 같은 경우에는 `limit`을 이용해서 명시적으로 스트림 크기를 제약해야 한다. 그렇지 않는다면 무한 스트림의 요소는 무한적으로 계싼이 반복되어 정렬하거나 리듀싱을 할 수 없다. 

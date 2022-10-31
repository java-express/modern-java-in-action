# Chapter 6 : 스트림으로 데이터 수집

#### collect() / Collector / Collectors

-   collect()

```
  List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
  List<String> result;

  result = list.stream().map(String::toUpperCase).collect(ArrayList::new,List::add,List::addAll);
  result = list.stream().map(String::toUpperCase).collect(Collectors.toList());

  // sysout(result) 결과 : [A, B, C]
```

collect()는 요소를 수집하는 기능을 합니다.

`collect(Supplier supplier, BiConsumer accumulator, BiConsumer combiner)`와 `collect(Collector collector)`가 있습니다.

<br/>

- collect(Supplier, BiConsumer, BiConsumer)

```
list.stream().collect(ArrayList::new, List::add, List::addAll);

//
collect(Supplier supplier, BiConsumer accumulator, BiConsumer combiner) {
    R result = supplier.get();

    for (T element : this stream)
        accumulator.accept(result, element);

    return result;
}
```

1.  supplier.get()(ArrayList::new)로 빈 누적자를 생성합니다.
2.  accumulator.accept()(List::add)로 스트림의 요소들을 처리합니다.
3.  누적결과를 반환합니다.

내부적으로 위 방식으로 동작하기 때문에 요소가 수집됨을 알 수 있습니다.

`collect(Collector collector)` 경우를 살펴보기 위해서는 `Collector 인터페이스`와 `Collectors 클래스`에 대하여 먼저 알아보는 것이 좋습니다.

<br/>

-   Collector 인터페이스의 시그니처

```
public interface Collector<T, A, R> {
    Supplier supplier();
    BiConsumer accumulator();
    BinaryOperator combiner();
    Function finisher();
    Set<Characteristics> characteristics();
}
```

collect()의 파라미터인 collector는 **Collector 인터페이스의 구현체**이며, **Collectors 클래스**의 `팩토리메서드`에 의해 생성됩니다.

1.  supplier / accumulator는 첫번째 collect메서드에서 확인했듯이, 각각 빈누적자 / 요소처리에 해당합니다.
2.  combiner는 병렬처리 시 서브파트를 병합할 때의 처리에 해당합니다.
3.  finisher는 스트림 탐색이 끝나고 최종 누적값을 (필요하다면) 변환하는 부분에 해당합니다.
4.  characteristics는 병렬처리 여부 또는 병렬 처리시 어떤 최적화를 선택할지 힌트를 제공합니다.

좀 더 자세히 알아보기 위하여, 교재에서 Collectors.toList()의 결과(Collector구현체)를 흉내낸 TolistCollector 클래스를 보도록 하겠습니다.

<br/>

```
//
list.stream().collect(new TolistCollector<String>());

//
class TolistCollector<T> implements Collector<T, List<T>, List<T>> { 
    @Override
    public Supplier<List<T>> supplier() { 
        return ArrayList::new;
    }
    @Override 
    public BiConsumer<List<T>, T> accumulator() { 
        return List::add; 
    }
    @Override 
    public Function<List<T>, List<T>> finisher() { 
        return Function.identity();
    }
    @Override 
    public BinaryOperator<List<T>> combiner() { 
        return (list1, list2) -> {
            list1 .addAll(list2);
            return list1; 
        }; 
    }
    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH, Characteristics.CONCURRENT));
    }
}
```

1. `collect(new TolistCollector<String>())`는 `collect(toList())`와 같이 동작합니다.
2. `supplier()`와 `accumulator()`의 경우, 첫 번째 collect메서드와 동일한 부분입니다.
3. `finisher()`는 최종 누적자의 변환부인데, toList()의 경우 누적자 그대로를 반환하면 되므로, identity()를 호출하는 모습을 볼 수 있습니다.
4. `combiner()`는 병렬처리 시 서브파트를 어떻게 합병할 것인지에 대한 부분입니다.
5. `characteristics()`는 여러개의 `Characteristics 열거형 값`을 리턴합니다.
6. characteristics()가 제공하는 열거형 힌트를 collector 객체를 파라미터로 사용하는 collect()같은 메서드에서 활용한다고 볼 수 있습니다.

[(링크)Chapter6. characteristics() 관련 이슈](https://github.com/java-express/modern-java-in-action/issues/16)

<br/>

#### 스트림 데이터 수집

```
for (Transaction transaction : transactions) {
    Currency currency = transaction.getCurrency();
    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
    if transactionsForCurrency == null) {
        transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
    }
    transactionsForCurrency.add(transaction);
}
```

위 레거시 코드를

```
transactions.stream().collect(groupingBy(Transaction::getCurrency));
```

위와 같이 간결하게 구현할 수 있음을 보여주면서 6장이 시작됩니다.

`transactions.stream().collect(groupingBy(Transaction::getCurrency));`

1.  groupingBy(Function classifier)메서드는 Collectors 팩토리 메서드로써, collector(Collector 구현객체)를 리턴합니다.
2.  파라미터로 받는 classifier의 결과값으로 (Map에 쓰일) 키(key)를 구성합니다.
3.  위 2번을 포함한 grouping 로직으로 Collector 추상메서드들을 구성한 후 CollectorImpl 객체(**특화된 컬렉터**)를 리턴합니다.

<br/>

```
  list.stream().collect(maxBy(comparingInt(String::length)));
  list.stream().collect(summinglnt(Dish::getCalories));
  list.stream().collect(joining());
```

중요한 것은 Collectors의 '어떤' 메서드와 / '어떤' 파라미터에 의해 만들어진 **특화된 컬렉터**를 이용한다는 것입니다.

<br/>

-   범용 reducing 팩토리 메서드

```
  list.stream().collect(reducing("", String:getName, String::concat)); // 초기값, 변환, 작업처리
  //list.stream().map(String::getName).collect(joining()); 와 같음
```

특화된 컬렉터를 제작하는 다른 팩토리 메서들과 달리 reducing을 이용하여 작업을 직접 지정해 줄 수 있습니다.

<br/>

-   gruopingBy에 collector를 받는 파라미터가 추가된 형태

```
list.stream().collect(groupingBy(String::length, filtering(String::isBlank, toList())));
list.stream().collect(groupingBy(String::length, mapping(str -> str.repeat(2), toList())));
list.stream().collect(groupingBy(String::length, groupingBy(String::length))); // 다수준 그룹화

list.stream().collect(groupingBy(String::length, collectingAndThen(maxBy(Comparator.comparingInt(String::length)), Optional::get)));
// list.stream().collect(groupingBy(String::length, maxBy(Comparator.comparingInt(String::length))를하면
// Map<Integer, Optional<String>>이므로 수집 후 처리(optional제거 등)를 하고플 때 사용하는 메서드

... // flatMapping, summing 등 있음
```

gruoping내부에 collector를 추가적으로 지정하여 다양하게 사용할 수 있습니다.

<br/>

-   조금 특별한 그룹화 **분할**

`list.stream().collect(partitioningBy(String::isEmpty, groupingBy(String::length)));`

키 값이 true/false로 지정되어 있는 grouping이라고 볼 수 있습니다.

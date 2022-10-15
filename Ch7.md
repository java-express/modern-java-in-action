### 들어가면서,

Java7 부터는 포크/조인 프레임워크(fork/join framework)을 통해 보다 쉽게 병렬화를 수행하면서 에러를 최소활할 수 있도록 해준다. 포크/조인 프레임워크와 내부적인 병렬 스트림 처리는 어떤 관계가 있는지 알아보자.
또한 커스텀 Spliterator를 직접 구현하면서 분할 과정을 원하는 방식으로 만들어보는 법도 함께 볼 것이다.

### 병렬 스트림

간단하게 컬렉션에 `paralleStream`을 호출하면 `병렬 스트림(parallel stream)`이 생성된다.
병렬스트림은, 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할했다.

### 스트림 성능 측정

- 반복형

```java
public long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i <= n; i++) {
        result += i;
    }

    return result;
}
```

- 순차 리듀싱

```java
public long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
            .limit(n)
            .reduce(0L, Long::sum);
}
```

- 병렬 리듀싱

```java
public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
            .limit(n)
            .parallel() // 스트림을 병렬 스트림으로 변환한다.
            .reduce(0L, Long::sum);
}
```

### JHM 라이브러리를 이용하여 성능 측정 해보기

## 병렬 스트림을 효과적으로 사용하기

- 확신이 서지 않으면 직접 측정하라. 적절한 벤치마크로 직접 성능을 측정하는 것이 바람직하다.
- 박싱을 주의하라
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다 (ex. 순서에 의존하는 연산)
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다
- 스트림을 구성하는 자료구조가 적절한지 확인하라 (ex. LinkedList보다 ArrayList가 효율적이다)
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다
- 최종 연산의 병합 과정 비용을 살펴보라

**참고**

- 모던자바인액션 7장
# [Chapter 15] CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

```
p.462

 요즘에는 독립적으로만 동작하는 웹사이트나 네트워크 애플리케이션을 찾아보기 힘들다.
 즉, 앞으로 만들 웹 애플리케이션은 다양한 소스의 콘텐츠를 가져와서
 사용자가 삶을 풍요롭게 만들도록 합치는 '매시업' 형태가 될 가능성이 크다.

 이런 애플리케이션을 구현하려면 인터넷으로 여러 웹 서비스에 접근해야 한다. 
 하지만 이들 서비스의 응답을 기다리는 동안 연산이 블록되거나
 귀중한 CPU 클록 사이클 자원을 낭비하고 싶진 않다.
 예를 들어 페이스북의 데이터를 기다리는 동안 트위터 데이터를 처리하지 말란 법은 없다. 
```

<br />

## 동시성 vs 병렬성

- [동시성과 병렬성 참고 링크](https://seamless.tistory.com/42)

<br />

## 15.1 동시성을 구현하는 자바 지원의 진화
- Runnable, Thread
- ExecutorService 인터페이스 (java 5)
- 포크/조인 프레임워크 (java 7)
- 람다 스트림 (java 8)
- 발행-구독 프로토콜 (java 9)

<br />

## 15.2 동기 API와 비동기 API : 메서드를 합하는 예제

- 스레드 이용 : 코드가 복잡해 진다.

```
main() {
	int X = 1337;
	Result result = new Result();

	Thread t1 = new Thread(() -> { result.left = f(x); });
	Thread t2 = new Thread(() -> { result.right = g(x); });

	t1.start();
	t2.start();
	t1.join();
	t2.join();

	System.out.println(result.left + result.right);

class Result {
	private int left;
	private int right;
}
```
<br />

- 스레드풀 & Future 이용 : 여전한 잡음

```
main() {
	int X = 1337;

	ExecutorService executorService = Executors.newFixedThreadPool(2);
	Future<Integer> y = executorService.submit(() -> f(x));
	Future<Integer> z = executorService.submit(() -> g(x));
	System.out.println(y.get() + z.get());

	executorService.shutdown();
```

<br />

- 리액티브(콜백 활용) 형식 + 별도의 결과 태스크 : 아직까지는 복잡함

```
main() {
	int X = 1337;
	Result result = new Result();

	f(x, (int y) -> {
		result.left = y;
		System.out_println((result.left + result.right));
	});

	g(x, (int z) -> {
		result.right = z;
		System.out.println((result.left + result.right));
	});
}
```

<br />

## 15.3/4 박스와 채널 모델 / CompletableFuture와 콤비네이터

![이미지](data:image/png;base64,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)

부분 작업을 통합하는 앞선 예제들을 위와 같이 표현할 수 있습니다.

15.2장의 예제 코드들의 문제를 CompletableFuture와 콤비네이터로 해결할 수 있습니다.

```
main() { 
	ExecutorService executorService = Executors.newFixedThreadPool(10); 
	int x = 1337;

	CompletableFuture<Integer> a = new CompletableFuture<>();
	CompletableFuture<Integer> b = new CompletableFuture<>();
	CompletableFuture<Integer> c = a.thenCombine(b, (y, z)-> Y + z);

	executorService.submit(() -> a.complete(f(x))); 
	executorService.submit(() -> b.complete(g(x)));

	System.out.println(c.get());
	executorService.shutdown();
}
```

<br />

## 15.5 발행-구독 프로토콜

CompletableFuture를 활용한 위 예제에서는 '한 번'만 실행함에 주목하여야 합니다.

웹 애플리케이션과 같이 응답을 기다리는 형태의 리액티브 시스템처럼

'여러 번'의 실행울 하는 경우가 필요하기 마련입니다.

![발행구독이미지](data:image/png;base64,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)

자바 9의 Flow 인터페이스의 발행-구독 모델(pub-sub)을 활용할 수 있습니다.

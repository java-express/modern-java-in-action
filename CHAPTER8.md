### ConcurrentHashMap
**ConcurrentHashMap**은 HashTable 클래스의 단점을 보완하면서 **Multi-Thread 환경**에서 사용할 수 있도록 나온 클래스이다.
ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한 다. 따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다(참고로, 표준 HashMap은 비동기로 동작함).

- ConcurrentHashMap.put()
1. 빈 해시 버킷에 노드를 삽입하는 경우, lock을 사용하지 않고 Compare and Swap을 이용하여 새로운 노드를 해시 버킷에 삽입한다.(원자성 보장)
```java
for (Node<K,V>[] tab = table;;) {
	Node<K,V> f; int n, i, fh;
    if (tab ==null || (n = tab.length) == 0)
    tab = initTable();
    else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    	if(casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
        break;	// no lock when adding to empty bin
    }
}
```

1) 무한 루프, table은 내부적으로 관리하는 가변 배열이다.
2) 새로운 노드를 삽입하기 위해, 해당 버킷 값을 가져와(tabAt 함수) 비어 있는지 확인한다.(==null)
3) 다시 Node를 담고 있는 volatile 변수에 접근하여 Node와 기대값(null)을 비교하여(casTabAt 함수) 같으면 새로운 Node를 생성해 넣고, 아니면 (1)번으로 돌아간다(재시도)
- volatile 변수에 2번 접근하는 동안 원자성(atomic)을 보장하기 위해 기대하는 값과 비교(Compare)하여 맞는 경우에 새로운 노드를 넣는다.(Swap)


2. 이미 노드가 존재하는 경우는 `synchronized(노드가 존재하는 해시 버킷 객체)`를 이용해 하나의 스레드만 접근할 수 있도록 제어한다.
```java
else {
	V oldVal = null;
	synchronized (f) {
		if (tabAt(tab, i) == f) {
			if (fh >= 0) {
				binCount = 1;
				for (Node<K,V> e = f;; ++binCount) {
					K ek;
					if (e.hash == hash &&((ek = e.key) == key ||(ek != null && key.equals(ek)))) {
						oldVal = e.val;
						if (!onlyIfAbsent)
							e.val = value;
						break;
					}
					Node<K,V> pred = e;
					if ((e = e.next) == null) {
						pred.next = new Node<K,V>(hash, key, value);
						break;
					}
				}
			}
		}
```

- 서로 다른 스레드가 같은 해시 버킷에 접근할 때만 해당 블록이 잠기게 된다.


>https://pplenty.tistory.com/17


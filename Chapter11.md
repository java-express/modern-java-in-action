# Chapter 11 : null 대신 Optional 클래스


## 🚨 null 때문에 발생하는 문제

- `nullPointerException` 자바에서 흔히 발생하는 에러이다.
- 코드를 어지럽힌다. 
null check를 하기 위해 중첩된 코드를 추가하므로 가독성이 떨어진다.
- 아무 의미가 없다.
null은 아무 의미도 표현하지 않는다. 정적 형식 언어에서 값이 없음을 표현하는 방법으론 적절하지 않다.
- 자바 철학에 위배된다.
자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터다.
- 형식 시스템에 구멍을 만든다.
null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 null이 어떤 의미로 사용되었는지 알 수 없다.

## 📌 그래서, null을 확인하기 위해 Optional 클래스를 사용한다.

### java.util.Optional<T>

- 자바 8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T>라는 클래스를 제공한다.

![image](https://user-images.githubusercontent.com/83346234/200340860-7503a99d-60d4-4fe2-a4d0-be599947ca6f.png)

값이 있으면 Optional 클래스는  값을 감싸고, 값이 없으면 Optional.empty 메서드로 빈값을 반환하기 때문에 NullPointerException에 대해 안전하도록 만들어 준다.

## Optional 클래스의 메서드

| 메서드 | 설명 |
| --- | --- |
| empty | 빈 Optional 인스턴스 반환 |
| filter | 값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional을 반환하고, 값이 없거나 프레디케이트와 일치하지 않으면 빈 Optional을 반환한다. |
| flatMap | 값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional을 반환하고, 값이 없으면 빈 Optional을 반환한다. |
| get | 값이 존재하면 Optional이 감싸고 있는 값을 반환하고, 값이 없으면 NoSuchElementException이 발생한다. |
| ifPresent | 값이 존재하면 지정된 Consumer를 실행하고 값이 없으면 아무 일도 일어나지 않음 |
| ifPresentOrElse | 값이 존재하면 지정된 Consumer를 실행하고 값이 없으면 아무 일도 일어나지 않음 |
| isPresent | 값이 존재하면 true를 반환하고 값이 없으면 false를 반환함 |
| map | 값이 존재하면 제공된 매핑 함수를 적용함 |
| of | 값이 존재하면 값을 감싸는 Optional를 반환, 값이 null이면 NullPointerException 발생 |
| ofNullable | 값이 존재하면 값을 감싸는 Optional를 반환, 갑싱 null이면 빈 Optional 반환 |
| or | 값이 존재하면 같은 Optional을 반환, 값이 없으면 Supplier에서 만든 Optional을 반환 |
| orEles | 값이 존재하면 값을 반환하고, 값이 없으면 기본 값을 반환 |
| orElseGet | 값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 생성한 값을 반환 |
| orElseThrow | 값이 존재하면 값을 반환, 값이 없으면 Supplier에서 생성한 예외 발생 |
| stream |  |

## 💡 orElse와 orElseGet의 차이점

- orElse() : 값이 null인 경우에도 메서드를 실행해야 할 때 사용

> → 메서드가 아는 값을 넘길 경우에는 `orElse()`를 사용하면 된다.
> 
- orElseGet() :  값이 null인 경우에만 메서드를 실행해야 할 때 사용

**참고**

- 모던 자바 인 액션 11장 - null 대신 Optional 클래스
- ****[[Java] Optional – orElse() vs orElseGet() 차이점 알고 쓰자.](https://kdhyo98.tistory.com/40)****

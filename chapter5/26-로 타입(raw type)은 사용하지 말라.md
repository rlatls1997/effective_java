## 26. 로 타입(raw type)은 사용하지 말라

클래스와 인터페이스 선언에 타입 매개변수가 사용되면 이를 **제네릭 클래스**, 혹은 **제네릭 인터페이스**라고 한다.

예시로 List인터페이스는 원소의 타입을 나타내는 타입 매개변수 E를 받는다.

제네릭 클래스와 제네릭 인터페이스를 통틀어서 **제네릭 타입**이라고 한다.

각 제네릭 타입은 일련의 **매개변수화 타입**을 정의한다.

예로 List<String>은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입이다.

여기서 String은 정규(formal)타입 매개변수 E에 해당하는 실제(actual)타입 매개변수다.

제네릭 타입을 하나 정의하면 그에 딸린 **로타입(raw type)**도 함께 정의된다.
로 타입은 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

예로List<E>의 로 타입은 List이다. 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진것처럼 동작하는데, 제네릭이 도래하기 전 코드와 호환되도록 하기 위한 방책이다.

**제네릭 지원 이전의 컬렉션 선언**

제네릭 지원이전에는 다음과 같이 컬렉션을 선언했다

```java
// Stamp인스턴스만 사용하는 컬렉션입니다.
private final Collection stamps = ...;
```

이 컬렉션은 실수로 Stamp외의 다른 타입의 객체를 넣어도 오류없이 컴파일된다.

다른 타입의 객체를 꺼내기 전까지는 오류를 알아채지 못한다.

```java
for(Iterator i = stamps.iterator(); i.hasNext();){
	Stamp stamp = (Stamp) i.next(); // ClassCastExceptiuon
	stamp.cancel();
}
```

오류는 가능한 발생 즉시에 발견하는 것이 좋다.

매개변수화된 컬렉션 타입으로 타입 안전성을 확보할 수 있다.

```java
private final Collection<Stamp> stamps = ...;
```

이제 컴파일러는 stamps 컬렉션에 Stamp인스턴스가 들어가야 함을 인지하게 된다.
따라서 다른 타입 인스턴스를 넣으면 컴파일단계에서 에러가 발생하여 잘못된 소스를 잡아준다.

또한 원소를 꺼내는 곳에서 보이지 않는 형변환을 추가하여 실패하지 않음을 보장한다.

따라서 로 타입을 써선 안된다. **로 타입의 사용은 제네릭이 안겨주는 안전성과 표현력을 잃게 만든다**

로 타입은 레거시코드의 호환성때문에 남아있는 것임을 기억하자.

List같은 로 타입의 사용안 안되나 List<Object>처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다. 차이는 List는 제네릭 타입에서 완전히 발을 뺀 것이고 List<Object>는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달했다는 점이다.

**비한정적 와일드카드 타입**

제네릭 타입을 사용하고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고싶지 않을 때 ?를 사용하자

제네릭타입인 Set<E>의 비한정적 와일드카드 타입은 Set<?>이다. 이것이 어떤 타입이라도 담을 수 있는 가장 범용적인 매개변수화 Set타입이다.

ex)

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2){..}
```

비한정적 와일드카드 타입인 Set<?>과 로 타입인 Set의 차이는??

⇒ 와일드카드 타입은 안전하고 로 타입은 안전하지 않다.

로타입 컬렉션에는 아무 원소나 넣을 수 있으니 타입 불변식을 훼손하기 쉽다. 반면 **Collection<?>에는 null외에는 어떤 원소도 넣을 수 없다.**

비한정적 와일드카드는 어떤 타입이 올지 모르기 때문에 타입이 존재하는 값을 넣을 수 없게 막는다.
즉, 컬렉션의 타입 불변식을 훼손하지 못하게 막았다. 컬렉션에서 꺼낼 수 있는 객체의 타입도 알 수 없다.

이런 제약이 너무 좁다면 제네릭 메서드나 한정적 와일드카드 타입을 사용하자.

**로타입을 쓰지 말라는 규칙**

- class 리터럴에는 로 타입을 사용해야 한다
  자바 명세는 class리터럴에 매개변수화 타입을 사용하지 못하게 했다.
  ex) List.class, int.class는 허용, List<String>.class는 허용 안함
- 런타임에는 제네릭 타입 정보가 지워지므로 instanceof연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 수 없다. 로타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작하므로 깔끔한 코드를 위해 로 타입을 쓰는 편이 좋다.

ex) 로 타입을 써도 좋은 예

```java
if(o instanceof Set){
	Set<?> s = (Set<?>) o;
}
```

instanceof에선 로타입을 썼다가도 검수 후엔 와일드카드타입인 Set<?>로 형변환해야한다. 이는 검사 형변환(checked cast)이므로 컴파일러 경고가 발생하지 않는다.

**결론**

로타입을 사용하면 안된다. 로타입은 안전하지 않다.

로타입의 존재 이유는 제네릭도입 전 코드의 호환성을 위함이다.

**용어정리**

- 매개변수화 타입 : List<String>
- 실제 타입 매개변수 : String
- 제네릭 타입 : List<E>
- 정규 타입 매개변수 : E
- 비한정적 와일드 카드 타입 : List<?>
- 로 타입 : List
- 한정적 타입 매개변수 : <E extends Number>
- 재귀적 타입 한정 : <T extends Comparable<T>>
- 한정적 와일드카드 타입 : List<> extends Number>
- 제네릭 메서드 : static <E> List<E> asList(E[] a)
- 타입 토큰 : String.class
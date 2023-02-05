# Chapter 11 - null 대신 Optional 클래스
In computing, a null pointer or null reference is a value saved for indicating that the pointer or reference does not refer to a valid object  
널 포인터 또는 널 참조는 포인터 또는 참조가 유효한 객체를 참조하지 않음을 나타내기 위해 저장된 값이다.  
```java
public String getCarInsuranceName(Person person) {
  return person.getCar().getInsurance().getName();
}
```
런타임에 NPE가 발생한다.  
  
모든 변수가 null인지 의심하면 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다.  
따라서 이와 같은 반복 패턴(recurring pattern) 코드를 깊은 의심(deep doubt)이라고 부른다.  
  
early- return : 너무 많은 출구  
## null 떄문에 발생하는 문제  
- 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러이다.  
- 코드를 어지럽힌다 : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.  
- 아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.  
- 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터다.  
- 형식 시스템에 구멍을 만든다 : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 포함할 수 있다. 시스템의 다른 부분으로 null이 퍼졌을 때 어떤 의미로 사용되었는지 알 수 없다.  
  
## java.util.Optional<T>
Java8은 '선택형값' 개념의 영향을 받아서 `Optional<T>`라는 클래스를 제공한다. 선택형값 = optional value  
  
Optional은 선택형값(optional value)을 캡슐화하는 클래스이다.  
값이 있으면 Optional 클래스는 값을 감싼다. 반면 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.  
Optional.empty()는 Optional의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드이다.  
  
null을 참조하려 하면 NPE가 발생하지만 Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다.  
```java
public class Person {
	private Optional<Car> car; // 사람이 차를 소유했을 수도 소유하지 않았을 수도 있으므로 Optional로 정의한다.

	public Optional<Car> getCar() {
		return car;
	}
}
```
Optional을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다.  
모든 null 참조를 Optional로 대치하는 것은 바람직하지 않다.  
  
## Optional 적용 패턴
Optional 형식을 이용해서 도메인 모델의 의미를 더 명확하게 만들 수 있었으며 null 참조 대신 값이 없는 상황을 표현할 수 있음을 확인했다.  
실제로는 Optional을 어떻게 활용할 수 있을까?  
  
### [빈 Optional] : Optional.empty로 빈 Optional 객체를 얻을 수 있다.  
```java
Optional<Car> optCar = Optional.empty();
```
### [null이 아닌 값으로 Optional 만들기] : Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.  
```java
Optional<Car> ofCar = Optional.of(car);
```
이제 car가 null이면 즉시 NPE 발생한다.(Optional을 사용하지 않았다면 car의 프로퍼티에 접근하려 할 때 에러가 발생했을 것이다.)  
### [null값으로 Optional] 만들기 : Optional.ofNullable로 null값을 저장할 수 있는 Optional을 만들 수 있다.
```java
Optional<Car> ofNullableCar = Optional.ofNullable(car);
```
car가 null이면 빈 Optional 객체가 반환된다.  
  
Optional이 비어있으면 get을 호출했을 때 예외가 발생한다. 즉, Optional을 잘못 사용하면 결국 null을 사용했을 때와 같은 문제를 겪을 수 있다.  
  
## Optional에서 값 가져오기
```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```
Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각할 수 있다.  
Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다. Optional이 비어있으면 아무 일도 일어나지 않는다.  
  
```java
public String getCarInsuranceName(Person person) {
  return person.getCar().getInsurance().getName();
}
```
맨 처음으로 본 다음과 같은 메서드를 어떻게 활용할 수 있을까? flatMap을 살펴보자.  
```java
public String getCarInsuranceName(Person person) {
		Optional<Person> optPerson = Optional.of(person);
		Optional<String> name = optPerson.map(Person::getCar)
			.map(Car::getInsurance)
			.map(Insurance::getName);
		return name.orElse("UnKnown");
	}
```
getCar가 `Optional<Car>`형식이므로 map 연산의 결과는 `Optional<Optional<Car>>` 형식이다.  
  
스트림의 flatMap은 함수를 인수로 받아서 다른 스트림을 반환하는 메서드이다.  
보통 인수로 받은 함수를 스트림의 각 요소에 적용하면 스트림의 스트림이 만들어진다.  
하지만 flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.  
즉, 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다.  
  
이차원 Optional을 일차원 Optional로 평준화해야 한다.  
  
이를 실제로 사용한 코드는 다음과 같다.  
```java
public String getCarInsuranceName(Optional<Person> person) {
		return person.flatMap(Person::getCar)
			.flatMap(Car::getInsurance)
			.map(Insurance::getName)
			.orElse("UnKnown");
	}
```
예를 들어 id로 사람을 검색했는데 id에 맞는 사람이 없을 수 있다. 따라서 Person 대신 `Optional<Person>`을 사용하도록 메서드 인수 형식을 바꿨다.  
  
Person을 Optional로 감싼 다음에 flatMap(Person::getCar)를 호출했다. 첫 번째 단계에서는 Optional 내부의 Person에 Function을 적용한다.(getCar 메서드)  
getCar 메서드는 `Optional<Car>`를 반환하므로 Optional 내부의 Person이 `Optional<Car>`로 변환되면서 중첩 Optional이 생성된다.  
따라서 flatMap 연산으로 Optional을 평준화한다.  
  
평준화 과정이란 이론적으로 두 Optional을 합치는 기능을 수행하면서 둘 중 하나라도 null이면 빈 Optional을 생성하는 연산이다.  
flatMap을 빈 Optional에 호출하면 아무 일도 일어나지 않고 그대로 반환된다. 반면 `Optional<Person>` 이라면 flatMap에 전달된 Function이 Person에 적용된다.  
Function을 적용한 결과가 이미 Optional이므로`(Optional<Car>)` flatMap 메서드는 결과를 그대로 반환할 수 있다.  
  
호출 체인 중 어떤 메서드가 빈 Optional을 반환한다면 전체 결과로 빈 Optional을 반환하고 아니면 관련 보험회사의 이름을 포함하는 Optional을 반환한다.  
Optional은 비어있을 때 기본값을 제공하는 orElse라는 메서드를 사용했다.  










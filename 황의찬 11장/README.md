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

## Optional 스트림 조작
Java9 에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream()메서드를 추가했다.  
Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능을 유용하게 활용할 수 있다.  
```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
		return persons.stream()
			.map(Person::getCar)
			.map(optCar -> optCar.flatMap(Car::getInsurance))
			.map(optIns -> optIns.map(Insurance::getName))
			.flatMap(Optional::stream)
			.collect(Collectors.toSet());
	}
```
첫 번째 map 변환을 수행하고 `Stream<Optional<Car>>`를 얻는다. 이어지는 두 개의 map 연산을 이용해 `Optional<Car>`를 `Optional<Insurance>`로 변환한 다음  
각각을 `Optional<String>`으로 변환한다. 
```java
Stream<Optional<String>> stream = ...
Set<String> result = stream.filter(Optional::isPresent)
                           .map(Optional::get)
                           .collect(toSet());
```
stream() 메서드를 이용하면 Optional을 0개 이상의 항목을 포함하는 스트림으로 변환한다.  
flatMap으로 전달해 두 수준인 스트림의 스트림으로 변환하고 다시 한 수준인 평면 스트림으로 바꿀 수 있다.  
  
- `get()` : 래핑된 값이 있으면 해당 값을 반환하고 값이 없으면 NoSuchElementException을 발생시킨다. 결국 중첩된 null 확인 코드를 넣는 상황과 크게 다르지 않다.  
- `orElse(T other)` : Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.  
- `orElseGet(Supplier<? extends T> other)` : Optional에 값이 없을 때만 Supplier가 실행된다. orElse의 게으른 버전. 디폴트 값을 만드는데 시간이 걸리거나 Optional이 비어있을 때만 기본값을  
생성하고 싶다면(기본값이 반드시 필요한 상황) orElseGet을 사용해야 한다.  
- `orElseThrow(Supplier<? extends X> exceptionSupplier)` : 는 Optional이 비어있을 때 예외를 발생시킨다는 점에서 get 메서드와 비슷하다. 하지만 이 메서드는 발생시킬 예외의 종류를 선택할 수 있다.  
- `ifPresent(Consumer<? super T> consumer)` : 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다. 값이 없으면 아무 일도 일어나지 않는다.  
  
Java9 에서는 다음의 인스턴스 메서드가 추가되었다.  
- `ifPresentOrElse(Consumer<? super T> action. Runnable emptyAction)` : 이 메서드는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent와 다르다.  
  
Person과 Car 정보를 이용해서 가장 저렴한 보험료를 제공하는 보험회사를 찾는 몇몇 복잡한 비즈니스 로직을 구현한 외부 서비스가 있다고 가정하자.  
```java
private Insurance findCheapestInsurance(Person person, Car car) {
		Insurance cheapestCompany = new Insurance();
		// ...
		return cheapestCompany;
	}
```
두 Optional을 인수로 받아서 null safe 버전의 메서드는 다음과 같이 구현할 수 있다.  
```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
		if (person.isPresent() && car.isPresent()) {
			return Optional.of(findCheapestInsurance(person.get(), car.get()));
		} else {
			return Optional.empty();
		}
	} 
```
어떤 조건문을 사용하지 않고 nullSafeFindCheapestInsurance() 메서드를 재구현할 수 있다.  
```java
public Optional<Insurance>nullSafeFindCheapestInsuranceQuiz(Optional<Person> person, Optional<Car> car) {
		return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```
첫 번째 Optional(person)이 비어있다면 인수로 전달한 람다 표현식이 실행되지 않고 그대로 빈 Optional을 반환한다.  
person 값이 있으면 flatMap 메서드에 필요한 `Optional<Insurance>`를 반환하는 Function의 입력으로 person을 사용한다.  
  
person과 car이 모두 존재하면 findCheapestInsurance 메서드를 안전하게 호출할 수 있다.  
  
보험회사 이름이 'CambridgeInsurance'인지 확인해야 한다고 가정.  
```java
Insurance insurance = ...;
if (insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
  // Do something...
}
```
다음과 같이 코드를 재구현할 수 있다.  
```java
public void filteringWithOptional(Optional<Insurance> optInsurance) {
		optInsurance.filter(insurance -> 
			"CambridgeInsurance".equals(insurance.getName()))
			.ifPresent(x -> System.out.println("ok"));
	}
```
filter 메서드는 프레디케이트를 인수로 받는다. Optional 객체가 값을 가지며 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고  
그렇지 않으면 빈 Optional 객체를 반환한다.  
Optional은 최대 한 개의 요소를 포함할 수 있는 스트림과 같다.  
  
프레디케이트 적용 결과가 true면 Optional에는 아무 변화도 일어나지 않는다. 하지만 결과가 false면 값은 사라져버리고 Optional은 빈 상태가 된다.  
  
## Optional을 사용한 실용 예제
### 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기
예를 들어 Map의 get 메서드는 요청한 키에 대응하는 값을 찾지 못했을 때 null을 반환한다. get 메서드의 시그니처는 고칠 수 없지만 get 메서드의 반환값은  
Optional로 감쌀 수 있다.  
```java
Object value = map.get("key");
```
문자열 key에 해당하는 값이 없으면 null이 반환될 것이다. map에서 반환하는 값을 Optional로 감싸서 이를 개선할 수 있다.  
```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```
### 예외와 Optional 클래스
값을 제공할 수 없을 때 null 반환하는 대신 예외를 발생시킬 때도 있다. 전형적인 예가 Integer.parseInt(String).  
작은 유틸리티 메서드를 구현해서 Optional을 반환할 수 있다.  
```java
public static Optional<Integer> stringToInt(String input) {
		try {
			return Optional.of(Integer.parseInt(input));
		} catch (NumberFormatException e) {
			return Optional.empty();
		}
	}
```
Optional의 최대 요소 수는 한 개이므로 Optional에서는 기본형 특화 클래스로 성능을 개선할 수 없다. 또한 flatMap, map, filter 등을 지원하지 않으므로 사용할 것을 권장하지 않는다.  




















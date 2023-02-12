## null 대신 Optional 클래스
  자바로 프로그램을 개발하면서 가장 자주 겪는 예외가 NullPointerException이라는 것은 거의 대부분의 개발자들이 동의할 수 있을것이다. 이제 이 예외를 줄이기 위한 다양한 방법들을 살펴보자.

### 값이 없는 상황을 어떻게 처리할까?
  다음처럼 자동차, 보험, 사람 객체를 중첩 구조로 구현했다고 가정하자.
```java
public class PersonV1 {
  private CarV1 car;
  public CarV1 getCar() {
    return car;
  }
}

public class CarV1 {
  private Insurance insurance;
  public Insurance getInsurance() {
    return insurance;
  }
}

public class Insurance {
  private String name;
  public String getName() {
    return name;
  }
}
```
다음 코드에서 어떤 문제가 발생할까?
```java
  public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
  }
```
만약 차를 소유하지 않은 사람이 존재하는 상황에 getCar를 호출하게 된다면 NullPointerException이 발생하게 될것이다. 만약 보험이 없다면? 사람이 존재하지 않다면? 마찬가지로 같은 에러를 마주치게 될것이다. 이를 방지하기 위해 아래와 같이 처리를 하는 방법이 있다:
```java
  public String getCarInsuranceNameNullSafeV1(PersonV1 person) {
    if (person != null) {
      CarV1 car = person.getCar();
      if (car != null) {
        Insurance insurance = car.getInsurance();
        if (insurance != null) {
          return insurance.getName();
        }
      }
    }
    return "Unknown";
  }
```
  이 방식에서는 모든 변수가 null인지 의심하므로 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가한다. 따라서 이와 같은 반복 패턴 코드를 깊은 의심(deep doubt)이라고 부른다. 이를 반복하다보면 당연히 코드는 엉망이 되고 가독성도 현저히 떨어지게 된다.

#### null 때문에 발생하는 문제
  - 에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
  - 코드를 어지럽힌다 : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.
  - 아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
  - 자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터다.
  - 형식 시스템에 구멍을 만든다 : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알수 없다.

### Optional 클래스 소개
  자바 8은 java.util.Optional<T>라는 새로운 클래스를 제공한다. Optional은 선택형값을 캡슐화하는 클래스다. 값이 있으면 Optional 클래스는 값을 감싸고, 값이 없다면 Optional.empty 메서드로 Optional을 반환한다. null 참조와 Optional.empty()는 무엇이 다른지 궁금할 것이다. 의미상으로는 비슷하지만 실제로는 차이점이 많다, 우선 null을 참조하려하면 NullPointerException이 발생하지만 Optional.empty()는 Optional 객체이므로 이를 다양한 방식으로 활용할 수 있다. 위 차/보험/사람 객체를 재정의 해보자:
```java
public class Car {
  private Optional<Insurance> insurance;
  public Optional<Insurance> getInsurance() {
    return insurance;
  }
}
public class Insurance {
  private String name;
  public String getName() {
    return name;
  }
}
public class Person {
  private Optional<Car> car;
  private int age;
  public Optional<Car> getCar() {
    return car;
  }
  public int getAge() {
    return age;
  }
}
```
  Optional 클래스를 사용하면서 모델의 의미가 더 명확해졌다. 사람은 자동차를 소유했을수도 아닐수도 있으며 자동차는 보험에 가입되어 있을수도 아닐수도 있다는것을 명확히 표현하고있다. 또한 보험회사 이름은 String 형식으로 선언되어 있어, 보험회사는 반드시 이름을 가져야 함을 보여준다.

### Optional 적용 패턴
#### Optional 객체 만들기
  Optional을 사용하려면 Optional 객체를 만들어야 하고 다양한 방법으로 가능하다.

**빈 Optional**
정적 팩토리 메서드 Optional.empty로 빈 Optional 객체를 얻을 수 있다.
```java
  Optional<Car> optCar = Optional.empty();
```

**null이 아닌 값으로 Optional 만들기**
또는 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들 수 있다.
```java
  Optional<Car> optCar = Optional.of(car);
```
이제 car가 null이라면 즉시 NullPointerException이 발생한다.

**null값으로 Optional 만들기**
마지막으로 정적 팩토리 메서드 Optional.ofNullable로 null값을 저장할 수 있는 Optional을 만들 수 있다.
```java
  Optional<Car> optCar = Optional.ofNullable(car);
```
car가 null이면 빈 Optional 객체가 반환된다.

#### 맵으로 Optional의 값을 추출하고 변환하기
  보통 객체의 정보를 추출할 때 Optional을 사용하는 일이 많다. 예를 들어 보험회사의 이름을 추출한다고 가정하자. 이름 정보에 접근하기 전에 null인지 아닌지 아래와같이 확인해야한다.
```java
  String name = null;
  if(insurance != null) {
    name = insurance.getName();
  }
```
  이런 유형의 패턴에 사용할 수 있도록 Optional은 map 메서드를 지원한다.
```java
  Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
  Optional<String> name = optInsurance.map(Insurance::getName);
```
  Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다, Optional이 비어있으면 아무 일도 일어나지 않는다.

#### flatMap으로 Optional 객체 연결
  위에 있었던 예제를 아래와같이 재구현 해보자:
```java
  Optional<Person> optPerson = Optional.of(person);
  Optional<String> name = optPerson.map(Person::getCar) // (1)
      .map(Car::getInsurance) // (2)
      .map(Insurance::getName);
```
  하지만 안타깝게도 위 코드는 컴파일되지 않는다. 왜냐하면 getCar가 `Optional<Car>` 형식의 객체를 반환하기 때문이다. 즉 map 연산의 결과는 `Optional<Optional<Car>>` 형식의 객체가 되기 때문이다. 이 문제를 해결하기 위해 flatMap을 사용할 수 있다. flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다. 즉, 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다.

#### Optional로 자동차의 보험회사 이름 찾기
```java
  public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
        .flatMap(Car::getInsurance)
        .map(Insurance::getName)
        .orElse("Unknown");
  }
```

#### Optional 스트림 조작
  자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다. Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능을 유용하게 활용할 수 있다. 아래는 사람 목록을 이용해 가입한 보험 회사 이름을 찾는 코드이다:
```java
  public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
        .map(Person::getCar)
        .map(optCar -> optCar.flatMap(Car::getInsurance))
        .map(optInsurance -> optInsurance.map(Insurance::getName))
        .flatMap(Optional::stream)
        .collect(toSet());
  }
```
  보통 스트림 요소를 조작하려면 변환, 필터 등의 일련의 체인이 필요한데 이 예제는 Optional로 값이 감싸있으므로 이 과정이 조금 더 복잡하다.

#### 디폴트 액션과 Optional 언랩
  - get()은 값을 읽는 가장 간단한 메서드면서 동시에 가장 안전하지 않은 메서드이다.
  - erElse 메서드를 이용하면 Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.
  - orElseGet는 orElse 메서드에 대응하는 게으른 버전의 메서드다. Optional에 값이 없을 때만 Supplier가 실행되기 때문이다. orElse가 비어있을때만 기본값을 생성하고 싶다면 이 메서드를 활용하자.
  - orElseThrows는 Optional이 비어있을 때 예외를 발생시킨다는 점에서 get 메서드와 비슷하다. 하지만 이 메서드는 발생시킬 예외의 종류를 선택할 수 있다.
  - ifPresent를 이용하면 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.
  - ifPresentOrElse는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent와 다르다.

#### Optional 합치기
  이제 Person과 Car 정보를 이용해서 가장 저렴한 보험료를 제공하는 보험회사를 찾는 로직을 구현한 서비스가 있다고 가정하자:
```java
  public Insurance findCheapestInsurance(Person person, Car car) {
    // 다른 보험사에서 제공한 질의 서비스
    // 모든 데이터 비교
    Insurance cheapestCompany = new Insurance();
    return cheapestCompany;
  }
```
  두 Optional을 인수로 받아서 `Optional<Insurance>`를 반환하는 null 안전 버전의 메서드를 구현해야 한다고 가정하자. 인수로 전달한 값 중 하나라도 비어있으면 빈 `Optional<Insurance>`를 반환한다. isPresent를 활용하면 아래와 같이 구현할 수 있다:
```java
  public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPresent()) {
      return Optional.of(findCheapestInsurance(person.get(), car.get()));
    } else {
      return Optional.empty();
    }
  }
```
  하지만 이 방법은 null 확인 코드와 크게 다를것이 없다, 그래서 map 과 flatMap을 활용하여 아래와 같이 구현하는 것이 가능하다.
```java
  public Optional<Insurance> nullSafeFindCheapestInsuranceQuiz(Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
  }
```

#### 필터로 특정값 거르기
  종종 객체의 메서드를 호출해서 어떤 프로퍼티를 확인해야 할 때가 있다. 예를 들어 보험회사 이름이 특정 이름과 일치하는지 확인해야 한다고 가정하자. 이 작업을 filter 메서드를 이용해서 다음과 같이 구현할수있다:
```java
  optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
              .ifPresent(x -> System.out.println("ok"));
```
  filter 메서드는 프레디케이트를 인수로 받는다. Optional 객체가 값을 가지며 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다. 그러므로 Optional이 비어있다면 filter 연산은 아무 동작도 하지 않고 값이 있다면 그 값에 프레디케이트를 적용한다. 프레디케이트 적용 결과가 true면 Optional에는 아무 변화도 일어나지않고 false면 값은 사라져버리고 Optional은 빈 상태가 된다.
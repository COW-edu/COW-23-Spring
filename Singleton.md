대부분의 스프링 어플리케이션 → 웹 어플리케이션

과거의 순수 DI 컨테이너는 호출될 때마다 객체를 새로 생성해서 메모리 낭비가 심했음

해결방안: 해당 객체가 딱 1개만 생성되고 공유하도록 설계하면 된다 → 싱글톤 패턴

### 싱글톤 패턴

클래스의 인스턴스가 딱 1개 생성되는 것을 보장하는 디자인 패턴.

2개 이상 인스턴스 생성 못하도록 막아야 함

static 영역에 객체를 딱 1개만 생성한다

객체 인스턴스를 조회하는 것은 public으로 선언

생성자는 private으로 선언하여 외부에서 new 키워드를 사용해 객체 생성을 못 하게 막는다

### Question: Assertions의 isSameAs 와 isEqualTo의 차이?

isSameAs: 인스턴스 비교

isEqualTo: 두 객체가 동일한 메모리 주소를 가지고 있는지 확인

즉 두 객체가 동일한 값을 가지지만 메모리에서 동일한 주소가 아닐 경우 False를 반환 → 개체 ID 테스트

isSameAs: 메모리에서 동일한 주소를 가진 객체인지의 여부에 관계없이 두 객체가 동일한 값을 갖는지 확인함.

객체의 값만 같다면 주소(레퍼런스)가 틀려도 True 반환

싱글톤 패턴을 구현하는 방법은 정말 많다. 강의에서는 객체를 미리 생성해두는 단순하고 안전한 방법 택함

다른 방법? 객체를 호출할 때 객체가 생성되어 있지 않으면 그때 생성해주는 방법 등이 있음

```java
public class Singleton {
// Private static variable to hold the single instance of the class
	private static Singleton instance;
// Private constructor to prevent instantiation from outside the class
	private Singleton() {
    // initialization code here
	}

// Public static method to get the single instance of the class
	public static Singleton getInstance() {
    // Create the instance if it doesn't exist yet
	    if (instance == null) {
	        instance = new Singleton();
	    }

    // Return the single instance of the class
	    return instance;
	}

// Other methods and variables of the class here
// ...
}
```

문제점

- 구현하는 코드 자체가 많이 들어감
- DIP 위반 → 클라이언트가 구체 클래스에 의존
- 클라이언트가 구체클래스에 의존해서 OCP 원칙 위반 가능성이 높음
    - 싱글톤에 의해 공유하는 객체의 변경이 일어나면 수정할게 많아진다
- 내부 속성 변경, 초기화 어려움
- private 생성자로 자식 클래스 생성이 어려움
- 유연성 X

### 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도 객체 인스턴스를 싱글톤으로 관리한다. → 싱글톤 레지스트리

싱글톤 패턴을 위한 코드 필요 X

### 싱글톤 방식의 주의점

싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 방식으로 생성한 객체는 상태를 유지(Stateful)하게 설계하면 안 된다.

→ Stateless 설계 필요

### 무상태성Stateless이란?

말 그대로 상태를 유지하지 않는 것인데, 주로 클라이언트-서버 관계에서 서버가 클라이언트의 상태를 보존하지 않는 것을 뜻한다.

<img width="766" alt="스크린샷 2023-04-15 오전 12 06 43" src="https://user-images.githubusercontent.com/104254012/232083246-d207b973-6443-43d9-91c4-45fb74a849aa.png">
<img width="782" alt="스크린샷 2023-04-14 오후 11 35 44" src="https://user-images.githubusercontent.com/104254012/232083372-58cbc08d-864e-49ae-812d-13f129a05211.png">
<img width="652" alt="스크린샷 2023-04-14 오후 11 32 25" src="https://user-images.githubusercontent.com/104254012/232083423-ae55c811-cdba-437e-9b89-392ab9f565db.png">

서버는 이전 클라이언트의 요청을 유지하지 않아도 클라이언트에서 정보를 하나하나씩 보내주기 떄문에 통신하는 데 문제는 없다. 싱글톤 방식도 마찬가지로 Stateless 설계를 적용할 수 있다.

- 특정 클라이언트에 의존적이거나 값을 변경할 수 있는 필드 있으면 X
- 필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용

스레드 A, B에서 사용자 A는 주문금액 만원을 price에 넣은 후 B가 2만원을 설정했을 때, 서로 price를 공유하게 되면 A의 주문금액이 2만원이라고 나온다.

이런 문제로 스프링 빈은 항상 무상태로 설계해야 함

### @Configuration, 바이트 코드 조작

스프링 컨테이너는 CGLIB 라는 바이트코드 조작 라이브러리를 사용하여 AnnotationConfigApplication에 파라미터로 넘긴 클래스와 클래스명이 같은 임의의 다른 클래스를 만들어서 그 다른 클래스를 스프링 빈으로 등록한다.

임의의 다른 클래스는 바이트코드가 조작되어 작성되어 있을 것이고, 싱글톤이 보장되도록 해준다. 

만약 @Configuration을 뺀다면, 바이트코드가 조작되지 않는다.

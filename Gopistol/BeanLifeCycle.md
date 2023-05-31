## 빈 생명주기 콜백

스프링 빈은 **객체 생성 → 의존관계 주입**의 라이프사이클을 가짐

<img width="399" alt="스크린샷 2023-05-09 오후 1 53 03" src="https://user-images.githubusercontent.com/104254012/237054498-e9ac8e17-03c7-4ccc-ba09-519e987c941f.png">

 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해 초기화 시점을 알려주는 다양한 기능을 제공하고, 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 줌으로써 안전한 종료 작업을 진행할 수 있음

여기서 **콜백이란?**

- 다른 함수의 인자로써 이용되는 함수
- 어떠한 이벤트에 의해 호출되는 함수

**빈 생명주기 콜백의 역할**

위 콜백 함수의 정의 중 두 번째 정의의 관점에서 보았을 때 빈 생명주기 콜백은 의존관계 주입이라는 이벤트에 의해 호출되어서 초기화 시점을 알려주고, 스프링 컨테이너 종료 전에 소멸 콜백이 호출되어서 종료 시점을 알려주는 역할을 한다고 볼 수 있다.

- URL 네트워크 연결 예제
    
    ```java
    package hello.core.lifecycle;
    
    public class NetworkClient {
    
        private String url;
    
        public NetworkClient() {
            System.*out*.println("생성자 호출, url = " + url); 
    	      connect();        
    				call("초기화 연결 메시지");   
    		}    
    
    		public void setUrl(String url) {
            this.url = url;   
    		}   
    	 //서비스 시작시 호출    
    		public void connect() {  
    	      System.*out*.println("connect: " + url); 
    	  }   
     
    		public void call(String message) {    
    		    System.*out*.println("call: " + url + " message = " + message);
        }    
    
    		//서비스 종료시 호출   
    		public void disconnect() { 
    	      System.*out*.println("close: " + url);
        }
    
    }
    ```
    

위의 예제에서 객체를 생성하는 시점엔 url 값을 주입받지 못하고, 객체 생성 이후 setUrl()이 호출되어야 url이 존재하게 된다.

그렇다면, 생성자 `NetworkClient()` 안에서 url을 parameter로 받아서 초기화까지 같이 해 주면 되는 것 아닌가?

### ⇒ 객체를 생성하는 부분과 초기화하는 부분을 나누자

생성자: 필수 정보(parameter)를 받고, 메모리를 할당해 객체를 생성하는 책임을 가짐

초기화: 이렇게 생성된 값들을 활용해 외부 커넥션을 연결하는 등 무거운 동작을 수행함

→ 그러므로 생성자 안에서 무거운 초기화 작업을 같이 수행하는 것보다는 객체 생성부분과 초기화하는 부분을 명확히 나누는 것이 유지보수 측면에서 좋음.

**스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원함**

### 1. 인터페이스(InitializingBean, DisposableBean)

```java
    @Override
    public void afterPropertiesSet() throws Exception {
		connect();
		call("초기화 연결 메시지");
		}
		@Override
    public void destroy() throws Exception {
        disConnect();
    }
```

- InitializingBean 은 afterPropertiesSet() 메서드로 초기화를 지원한다.
- DisposableBean 은 destroy() 메서드로 소멸을 지원한다.

### 단점

- 스프링 전용 인터페이스로, 해당 코드가 스프링 전용 인터페이스에 의존함
- 메서드 이름 변경 불가
- 외부 라이브러리에 적용 불가

### 2. 설정 정보에 초기화 메서드, 종료 메서드 지정

`@Bean(initMethod = "init", destroyMethod = "close")`로 초기화, 소멸 메서드 지정 가능

특징

- 메서드 이름 자유롭게 사용 가능
- 스프링 빈이 스프링 코드에 의존하지 않음
- 코드 수정 불가능한 외부 라이브러리에도 초기화, 종료 메서드 적용 가능

> 라이브러리는 대부분 close , shutdown 이라는 이름의 종료 메서드를 사용한다. @Bean의 destroyMethod 는 기본값이 (inferred) (추론)으로 등록되어 있다. 이 추론 기능은 close , shutdown 라는 이름의 메서드를 자동으로 호출해준다. 따라서 직접 스프링 빈으로 등록하면 종료 메서드는 따로 적어주지 않아도 잘 동작한다. 추론 기능을 사용하기 싫으면 destroyMethod="" 처럼 빈 공백을 지정하면 된다.
> 

### 3. @PostConstruct, @PreDestroy 애노테이션 지원

```java
    @PostConstruct
    public void init() {
				System.out.println("NetworkClient.init");
				connect();
				call("초기화 연결 메시지");
		}
    @PreDestroy
    public void close() {
        System.out.println("NetworkClient.close");
        disConnect();
    }
```

특징

- 최신 스프링에서 가장 권장하는 방법
- 스프링에 종속적인 기술이 아니라 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 잘 동작함
- 외부 라이브러리엔 적용하지 못한다는 단점이 있다. 그럴 땐 두 번째 방법인 @Bean 기능 사용

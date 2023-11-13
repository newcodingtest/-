# 아이템 17. 변경 가능성을 최소화 해라

### 불변클래스,불변객체

- setter를 만들지 않는다.
- 상속을 허용하지 않는다.
- 모든 필드를  final 로 선언한다
- 모든 필드를 private 로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

```
public final class Person {
	
	private final Address address;
	
	public Person(Address address){
		this.address = address;
	}
	
	//잘못된 방법
	public Address getAddress(){
		return this.address;
	}
	
	//개선된 방법 --> 방어적인 복사
	public Address getAddress(){
		Address copyOfAddress = new Address();
		copyOfAddress.setStreet(address.getStreet());
		copyOfAddress.setZipCode(address.getZipCode());
		copyOfAddress.setCity(address.getCity());
		return copyOfAddress;
	}
	
	public static void main(String[] args){
		Address address = new Address();
		address.setCity("Seoul");
		
		Person person = new Person(address);
		Address change = person.getAddress();
		
		//불변 클래스인 Person의 필드인 Address를 final 로 선언해도 Address 내부에 setter가 존재하면 변경이 가능하다. 이는 안좋은 방법이다.
		change.setCity("Suwon");
		
		//개선된 방법으로 적용했을 경우 Person의 갖고 있는 기존 Address의 Seoul 정보는 변경이 안된다.
		System.out.println(person.address.getCity());
	}
	
}


@Getter
@Setter
public class Address {
	private String zipCode;
	private String street;
	private city;
}
```



### 장점

- 멀티스레드 환경에서 공유해서 사용하더라도 변경이 되지 않기 때문데 안정적이다.
- 캐싱해서 사용하기 때문에 여러 스레드에서 한번만 인스턴스를 호출해서 사용한다. 이는 성능적으로 이점이다.
- 데이터의 원자성이 보장된다.



### 단점

- 새로운 인스턴스를 만드는것이 단점이다

```
	//개선된 방법 --> 방어적인 복사
	public Address getAddress(){
		Address copyOfAddress = new Address();
		copyOfAddress.setStreet(address.getStreet());
		copyOfAddress.setZipCode(address.getZipCode());
		copyOfAddress.setCity(address.getCity());
		return copyOfAddress;
	}
```

- 기존 객체를 변경해야 될 경우 단점일 수 있다.








### 불변 클래스 만들 때 고려사항

- 상속을 막을 수 있는 다른 방법

  - `클래스에 final`을 적용 하는 방법과 다른 하나는 `생성자`에 `private` 을 적용하는 것
  - 외부에서 인스턴스 사용시에는 정적 팩토리 메서드로 접근한다.

  ```
  public class Yoon {
  	private name;
  	private age;
  
  	private Yoon(String name, int age){
  		this.name = name;
  		this.age = age;
  	}
  	
  	// 정적 팩토리 메서드
  	public static Yoon valueOf(String name, int age){
  		return new Yoon(name, age);
  	}
  	
  	//패키지 내부에서는 상속 가능
  	private static class YoonJuYoung extends Yoon {
  		private YoonJuYoung(String name, int age){
              this.name = name;
              this.age = age;
  		}
  	}
  }
  ```

  - private 생성자를 사용하지만 패키지 내부에서 상속할 수 있기 때문에 유연하다.
  - 정적 팩토리 메서드를 사용하면 구체 클래스를 자유롭게 정할수 있기 때문에 유연하다. 외부에서는 상위 인스턴스로만 받기 때문에 캡슐화도 된다.

     ex) BigInteger 참고




### final과 자바 메모리 모델(JVM)

- 불안전한 초기화가 발생할 수 있다.

  ```
  public class Yoon {
   final int x;
   int y;
   
   public Yoon(){
   	this.x = 3;
   	this.y = 4;
   }
  }
  
  public class Client {
  	public static void main(String[] args){
  		Yoon y = new Yoon();
  	}
  }
  ```

  - `메모리 모델`은 `하나의 쓰레드` 안에서 안전한 초기화가 가능하지만 `멀티 쓰레드`인 경우 보장할 수 없다. 

  - JVM 모델에 따라서 초기화 순서가 다를 수 있는데

  ```
  Object y = new Yoon()
  yoon = y
  y.x = 3;
  y.y = 4;
  ```

  이런식으로 초기화 과정에서 `y.y = 4`라는 값이 세팅되기 전에 0인 상태에서 객체를 참조할경우 불안전한 초기화 문제가 발생할 수 있다.(멀티쓰레드 환경에서 발생 가능성이 존재)

  `this.x` 는 `3`을 보장하지만 `this.y` 는 `0`이 될수도 있다.

  `때문에 객체를 초기화할때 필드가 값을 무조건 배정받고 초기화 되기를 보장받고 싶다면 final 키워들 사용하자.`









### 병행 프로그래밍에 유용하게 사용할 수 있는 유틸리티 

- `병행`은 여러 작업을 번갈아 가며 실행하여 `마치 동시에 여러 작업을 동시에 처리하듯이 보이지만`, `실제로는 한번에 오직 한 작업만 실행한다.`(CPU가 `한개`여도 가능)

  ![image](https://github.com/newcodingtest/-/assets/57785267/ec75eb55-b520-47dc-90ce-4cec4abe95c2)


- `병렬`은 `여러 작업을 동시에 처리`한다.(CPU가 `여러개` 있어야 한다.) 

 ![image](https://github.com/newcodingtest/-/assets/57785267/74aa4de0-1367-489f-b8df-e773c6e1cc15)



- 유용한 유틸리티로는 `CountDownLatch` 가 있다.


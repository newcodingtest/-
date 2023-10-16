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




출처: https://www.inflearn.com/course/%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94-1/lecture/108741?tab=curriculum&volume=0.83

**강의를 보고 자료 정리 하였음**

## 아이템3. 생성자나 열거 타입으로 싱글턴임을 보증하라

#### [1번째 방법]

> private 생성자 + public static final 필드

```
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis();
}
```



- 장점

간결하고 싱글턴임을 API에 들어낼 수 있다.(javadoc 사용시 쉽게 파악이 가능)



- 단점

1. **싱글톤을 사용하는 클라이언트 코드를 테스트하기 어려워진다.**

   **[대안]**

>    인터페이스를 만들어서 해당 인터페이스를 상속하여 테스트 할려는 메소드를 오버라이딩 하자



2. 리플렉션으로 private 생성자를 호출할 수 있다.

​	**[대안]**

```
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private static boolean craeted;
	
	private Elvis(){
		if(created){
			throw new UnsupportedOperationException("can't be created by 				construct");
		}
		created = true;
	};
}
```



3. 역직렬화 할 때 새로운 인스턴스가 생길 수 있다.

​	**[대안]**

```
public class Elvis implement Serializable {
	public static final Elvis INSTANCE = new Elvis();
	private static boolean craeted;
	
	private Elvis(){
		if(created){
			throw new UnsupportedOperationException("can't be created by 				construct");
		}
		created = true;
	};
	
	//문법적으로는 이상하지만, 오버라이딩 처럼 동작한다.
	private Object readResolve(){
		return INSTANCE;
	}
}
```

**추신: 스프링에서는 Bean으로 등록하자!! 엄청 편함^^ 스프링이 알아서 해줌**



#### **[두번째 방법]**

> private 생성자 + 정적 팩토리 메서드

```
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();

	private Elvis();
	
	public static Elvis getInstance(){return INSTANCE;}
}
```

> 첫번째 방법과 비슷하지만, static 메서드를 통해서 인스턴스를 생성한다.
>
> 허나 첫번째 단점과 동일하다.

- 장점

1. 클라이언트 코드를 만지지 않고 내가 원하는 인스턴스를 유동적으로 메서드를 통해 조작(생성)해줄수있다.

```
public static Elvis getInstance(){return new Elvis();}
```



2. 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있다.(원하는 타입으로 변환해줌)

```
public class MetaElvis<T> {

    private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();

    private MetaElvis() { }

    @SuppressWarnings("unchecked")
    public static <E> MetaElvis<E> getInstance() { return (MetaElvis<E>) INSTANCE; }

    public void say(T t) {
        System.out.println(t);
    }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public static void main(String[] args) {
        MetaElvis<String> elvis1 = MetaElvis.getInstance();
        MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
        System.out.println(elvis1);
        System.out.println(elvis2);
        elvis1.say("hello");
        elvis2.say(100);
    }

}
```

>     MetaElvis<String> elvis1 = MetaElvis.getInstance();
>     MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
>
> 원하는 타입으로 형변환을 해줄수 있다는 장점이 있다.
>
> 그러나 같은 인스턴스 이지만 타입이 다르기 때문에 equals 비교는 true 지만, == 비교는 false이다.



3. 정적팩토리의 메서드 참조를 공급자로 사용할 수 있다.

```
public class Concert {

	//공급자 사용
    public void start(Supplier<Singer> singerSupplier) {
    	//singerSupplier는Singer의 인자 없는 메서드를 호출하여 무언가 오브젝트를 리턴한다.
        Singer singer = singerSupplier.get();
        singer.sing();
    }

    public static void main(String[] args) {
        Concert concert = new Concert();
        //메소드 참조
        concert.start(Elvis::getInstance);
    }
}

public class Elvis implements Singer {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();

        System.out.println(Elvis.getInstance());
        System.out.println(Elvis.getInstance());
    }

    @Override
    public void sing() {
        System.out.println("my way~~~");
    }
}

public interface Singer {

    void sing();
}
```





#### **[세번째 방법]** **(권장방법!!!!!!!!!!!)**

> 열거타입을 사용해라,  리플랙션 및 직렬화, 역직렬화 단점이 해결됨

```
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```

> 테스트 시, 인터페이스를 구현하여 사용하면 된다.

```
public enum Elvis implements Singer {
    INSTANCE;
    
    @Oiverride
    public void sing(){...}
    ...
    }
```
<br>

<br>

### 키워드 정리

#### -리플렉션 API

#### -메서드 참조

```
public class Person {
	
	LocalDate birthday;

	public Person() {
		
	}
	
	public Person(LocalDate birthday) {
		this.birthday = birthday;
	}
	
	public int compareByAge(Person t) {
		return this.birthday.compareTo(t.birthday);
	}
	
	public static void main(String[] args) {
		List<LocalDate> dates = new ArrayList<LocalDate>();
		dates.add(LocalDate.of(1996, 12, 30));
		dates.add(LocalDate.of(1996, 12, 30));
		dates.add(LocalDate.of(1996, 12, 30));
		
		//Function을 이용한 일반 람다식
		dates.stream().map(d ->{
			return new Person(d);
		}).collect(Collectors.toList());
		//Function을 이용한 일반 람다식 -> 메서드 참조로 변환
		dates.stream().map(Person::new).collect(Collectors.toList());


		List<Person> people = new ArrayList<Person>();
		
		people.add(new Person(LocalDate.of(1996, 12, 30)));
		people.add(new Person(LocalDate.of(1996, 11, 30)));
		people.add(new Person(LocalDate.of(1996, 10, 30)));
		
		Comparator<Person> comparedByAge = Person::compareByAge;
		
		people.sort(comparedByAge);
		
	}
}
```

> Person::new
>
> 메서드 참조로 코드가 더 간편해짐을 확인할수 있음

<br>

#### -함수형 인터페이스

```
public class FunctionalInterface {
	public static void main(String[] args) {
		Function<Integer, String> intToString = Object::toString;
		
		//메서드의 파라미터가 없지만 리턴 값만 있는것
		Supplier<Person> integerSupplier = Person::new;
	    Function<LocalDate,Person>personFunction = Person::new;
				
	    //리턴은 없는데 파라미터가 있는것 ex)syso, 먼가 받아서 출력만 하는것
		Consumer<Integer> integerConsumer = System.out::println;
		
		//Boolean을 리턴함, 주로 stream 에서 filter 안에서 사용됨
		Predicate<Integer> predicate;
	}
}
```

<br>

#### -직렬화 역직렬화

> 직렬화: 객체를 바이트스트림으로 변환
>
> 역직렬화: 바이트스트림을 객체로 변환

JVM이 아니라면 역직렬화를 못하기 때문에 보통 

범용적인 XML이나 JSON 포멧을 많이 사용한다.

<br>

##### 직렬화를 원하지 않는 필드가 있다면 transient를 필드에 추가해주자

> ex) String transient String unnessary; 

<br>

##### static 필드는 직렬화가 되지 않는다.

> why? 인스턴스 값이 아닌 클래스 값이기 때문에

<br>

##### 직렬화를 한 다음 클래스 정보가 변경(필드가 변경되거나 추가)된다면 역직렬화가 될까?

> SerialVersionUID 값이 달라지기 때문에 역직렬화가 안된다.

**[대안]**

**클래스 정보가 변경되었지 역직렬화 또는 직렬화를 하고싶다면**

> 클래스 필드에 SerialVersionUID 을 선언해줘야한다.
	


#### 심화

**Serializable 과 Externalizable의 차이는?**

출처:https://hashcode.co.kr/questions/780/externalizable%EB%8A%94-%EB%AD%94%EA%B0%80%EC%9A%94

Externalizable 인터페이스는 Serializable 인터페이스를 확장한 형태로 세부적인 필드를 직접 제어할 수 있는 특징이 있다. transient 키워드를 사용하여 특정 필드를 제외시키는 것도 가능하지만 좀 더 세부적인 작업이 필요할 때 Externalizable 인터페이스를 사용할 수 있다.

1. Externalizable 인터페이스 Externalizable 인터페이스를 사용하면 객체 스트림을 통하여 객체를 읽고 쓸 때 내부의 내용을 자유롭게 조작하면서 원하는 내용을 출력할 수 있도록 도와준다. 이를 위해서 이 인터페이스는 Serializable 인터페이스를 확장하고 두 가지 메소드를 추가하였다. 하나는 출력을 위한 writeExternal() 메소드이고 또 하나는 입력을 위한 readExternal() 메소드다.
2. 객체 스트림 직렬화시의 동작 (1) 출력 (직렬화)
3. 객체 스트림을 통하여 객체를 출력하는 메소드가 호출되면 가장 먼저 그 객체의 클래스가 Externalizable 인터페이스를 구현했는지 확인한다. 만약 구현했다면 writeExternal() 메소드를 통해서 전송 데이터를 직렬화 한다.
4. Externalizable 인터페이스를 구현하지 않았다면 Serializable 인터페이스를 구현했는지 확인한다. Serializable 인터페이스를 구현했다면 그대로 객체 스트림에 객체 전체를 직렬화 한다.
5. Externalizable, Serializable 인터페이스를 구현하지 않았다면 Exception이 발생한다.

(2) 입력 (역직렬화)

- 스트림을 통해 전송받은 객체가 Externalizable 인터페이스를 구현했는지 확인한다. 구현했다면 readExternal() 메소드를 통해서 전송받은 데이터를 순서대로 읽어온다.
- Externalizable 인터페이스를 구현하지 않았다면 Serializable 인터페이스를 통해 객체 전체를 역직렬화 한다.

(3) writeExternal(), readExternal()

- Externalizable 인터페이스를 구현할 때는 writeExtenral()과 readExternal() 메소드를 작성해야 한다.
- Serializable 인터페이스는 모든 변수(transient 제외)를 직렬화하는 반면 Externalizable 인터페이스는 직렬화할 대상을 직접 writeExternal() 메소드에 구현해야 한다. (transient 변수도 직렬화하여 전송할 수 있다.)
- out.writeObject(name); 과 같이 직렬화할 변수 하나하나 writeObject()에 써줘야 한다.
- 스트림을 통해 객체를 받아 역직렬화를 할 때에도 readExternal() 메소드에서 하나하나 받은 데이터를 변수에 넣어줘야 한다.
- in.readObject(); 를 사용하여, writeExternal() 메소드에서 writeObject()에 넣어 준 변수의 순서대로 데이터를 가져온다.	
	

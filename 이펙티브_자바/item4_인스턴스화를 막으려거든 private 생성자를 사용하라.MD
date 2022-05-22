### 아이템4_인스턴스화를 막으려거든 private 생성자를 사용하라

> UtilityClass 같은 클래스에 해당한다. ex)String

보통 String을 만들때 이런식으로 만들지는 않는다.

> String target = new String();

- 정적 메서드만 담은 유틸리트 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다
- 추상 클래스를 만드는 것으로만으론 인스턴스화를 막을 수 없다.
- private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
- 생성자에 주석으로 인스턴스화 불가한 이유를 설명하는 것이 좋다
- 상속을 방지할 때도 같은 방법을 사용할 수 있다.

```
public class UtilityClass {

	//절대로 인스턴화가 되면 안되는 상황 --> 에러로 알려주자
	/**
	 * 이 클래스는 인스턴스를 만들 수 없습니다.
	 */
	private UtilityClass(){
		throw new AssertionError();
	}
	
	public static String hello(){
		return "hello";
	}

}
```

> 스프링 같은 경우도 abstract 로 선언되있는것이 있지만 private 생성자가 없기때문에 public static 메소드에 바로 접근이 가능하다.
>
> ex) AnnotationConfigUtils

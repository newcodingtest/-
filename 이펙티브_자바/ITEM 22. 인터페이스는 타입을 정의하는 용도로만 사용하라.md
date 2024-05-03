## ITEM 22. 인터페이스는 타입을 정의하는 용도로만 사용하라.

상수를 정의하는 용도로 인터페이스를 사용하지 말 것

- 클래스 내부에서 사용할 상수는 내부 구현에 해당한다.
- 내부 구현을 클래스의 API로 노출하는 행위가 된다.
- 클라이언트에 혼란을 준다.



상수 타입 선언 방법

- 유틸리티 클래스를 만들어 준다.

```
public class Constants {
	private Constants() {} //인스턴스화 방지
	
	public static final String NOT_FOUND_RESPONSE_MSG = "찾을 수 없어";
	public static final String SERVER_ERROR_RESPONSE_MSG = "서버가 맛이 갔어";
}
```






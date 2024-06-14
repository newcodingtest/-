
아이템 16. Public 클래스 에서는 public 필드가 아닌 접근자 메서드를 사용하라<br>
**worst**
```
public class Yoon {
	public String name
	public String age
}    
```

- 클라이언트 코드에서 필드에 직접 접근할 수 있기 때문에 캡슐화가 깨진다.
- 필드 접근시 부수 작업을 할 수 없음 -> 유연한 작업을 할 수 없음
 + 필드를 가져와서 다른 값을 만든다던지, 검증을 한다던지 등의 작업을 못한다.<br>
![image](https://github.com/newcodingtest/leetcodeYJY/assets/57785267/92912c20-dfff-4bfd-ab31-ee7e1dfa8bb8)

**Best**
```
public class Yoon {
	private String name
	private String age

	public String getName(){
		return this.name;
	}
	
	public String setName(){
		return this.name;
	}
	...
}     

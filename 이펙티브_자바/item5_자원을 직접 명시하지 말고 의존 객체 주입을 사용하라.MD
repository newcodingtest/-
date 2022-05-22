
### 아이템5_자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 사용하는 자원에 따라 동작이 달라지는 클래스는 정적 유틸리티 클래스나 싱글턴 방식이 합하지 않다.
- 의존 객체 주입이란 인스턴스를 생성할 때 필요한 자원을 넘겨주는 방식이다.
- 이 방시의 변형으로 생성자에 자원 팩터리를 넘겨줄 수 있다.

- 클래스의 유연성 , 재사용성, 테스트 용이성을 개선할 수 있다.



**예제코드**

```
public class SpellChecker {
	
	private final Dictionary dictionary;
	
	public SpellChecker(Dictionary dictionary){
		this.dictionary = dictionary;
	}
	
	public SpellChecker(DictionaryFactory dictionaryFactory){
		this.dictionary = dictionaryFactory.get();
	}
	
	public boolean isValid(String word){
		//+SpellChecker code
		...
		return dictionary.contains(word);
	}
	
	public List<String> suggestions(String type){
		//+SpellChecker code
		...
		return dictionary.closeWordsTo(type);
	}
	
}
```





Supplier<T> 인터페이스로 팩터리를 표현할 수 있다.

```
	//변경전========================
	public SpellChecker(DictionaryFactory dictionaryFactory){
		this.dictionary = dictionaryFactory.get();
	}
	//변경후========================
	public SpellChecker(Supplier<Dictionary> dictionarySupplier){
		this.dictionary = dictionarySupplier.get();
	}
	
```



#### **팩터리 메소드 패턴**

> 구체적으로 어떤 인스턴스를 만들지는 서브 클래스가 정한다.

![팩토리 메소드 패턴(Factory Method Pattern)](https://t1.daumcdn.net/cfile/tistory/9999D8385BE5725009)



```
public class SpellChecker {
	private Dictionary dictionary;
	
	public SpellChecker(DictionaryFactory dictionaryFactory){
		this.dictionary = dictionaryFactory.getDictionary();
	}
	
	public boolean isValid(String word){
		//+SpellChecker code
		...
		return dictionary.contains(word);
	}
	
	public List<String> suggestions(String type){
		//+SpellChecker code
		...
		return dictionary.closeWordsTo(type);
	}

}
```



```
public Interface DictionaryFactory {

	Dictionary getDictionary();
}
```

```
public class MockDictionary Dictionary{

	@Override
	public boolean contains(String word){
		return false;
	}
	
	@Override
	public List<String> closeWordsTo(String type){
		return null;
	}
}
```



```
public class DefaultDictionaryFactory implements DictionaryFactory{

	@Override
	public Dictionary getDictionary(){
		return new DefaultFactory();
	}
}
```



```
public class MockDictionaryFactory implements DictionaryFactory{

	@Override
	public Dictionary getDictionary(){
		return new MockDictionary();
	}
}
```



> 위의 코드를 보면 새로운 팩토리 코드가 추가되어도 
>
> 클라이언트 에서는 코드의 변경없다.
>
> 다시 말해서 확장에 열려있고 변경에 닫혀있는 상태이다.



#### 스프링 IOC

수많은 개발자에게 검증되었으며, 자바 표준 스펙도 지원

손쉽게 싱글톤 scope을 사용할 수 있다.

객체 생성 관련 라이프사이클 인터페이스를 제공

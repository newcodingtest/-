## ITEM 21. 인터페이스는 구현하는 쪽을 생각해 설계하라.

기존 인터페이스에 디폴트 메서드를 추가하는 것은 위험한 일

- 디폴트 메서드는 `구현 클래스`에 대해 아무것도 모른 채 합의 없이 무장적 삽입될 뿐
- 디폴트 메서드는 기존 구현체에 **런타임 오류**를 일으킬 수 있다.



ex1) 만약 인터페이스의 디폴트 메서드에 삭제 메서드가 구현되어 있다. 그 하위 구현체들에는 멀티 스레드 환경에서 안정성을 보장하는 syncronized 메서드들이 구현되어 있는 상황이다. 이때 디폴트 메서드를 호출하면 syncronized 메서드에 **해를 끼칠 수 있는 동작이 발생**한다.(**ConcurrentModificationException 예외 발생**)



ex2) 동일 메서드가 `SuperClass`와 `SuperInterface `에 존재하는 상황이라면?

메서드 호출 법칙으로 `클래스` 는 `인터페이스`를 항상 이기기 때문에 클래스의 메서드가 호출될 것이다.

```java
public class SubClass extends SuperClass implements SuperInterface {
	...
}
```


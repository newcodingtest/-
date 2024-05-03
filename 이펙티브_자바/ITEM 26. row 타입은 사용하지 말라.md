## ITEM 26. row 타입은 사용하지 말라

매개변수화 타입을 사용하라.

```
public class Test<E> {
	...
}

Test test = new Test(); //x
Test<String> test = new Test(); //o
```

- 컴파일 타임에 문제를 찾을 수 있다.(안정성)

- `raw 타입`을 사용하면 안정성과 표현력을 잃는다.

  

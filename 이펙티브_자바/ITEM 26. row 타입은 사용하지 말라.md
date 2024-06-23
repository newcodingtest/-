## ITEM 26. raw 타입은 사용하지 말라

<br>

매개변수화 타입을 사용하라.

```
public class Box<E> {
	...
}

Box box = new Box(); //x , row 타입을 사용한것
Box<Integer> box = new Box(); //o , 직접 타입을 선언해서 사용하자 
```

<br>

- 컴파일 타임에 문제를 찾을 수 있다.(안정성)
- `raw 타입`을 사용하면 안정성과 표현력을 잃는다.

<br>

 `Box<Integer> box = new Box();` 에서 타입을 선언하든 선언하지 않든 컴파일된 파일정보 에서는 기본적으로 모두 `raw 타입(Object)`으로 적용되지만 `Test<String> test`로 타입을 선언하면 컴파일러가 `타입 형변환 코드`를 추가적으로 넣어준다.

<br>

바이트코드를 보면`Box<Integer> box = new Box();` 로 타입을 명시적으로 선언했을때도 컴파일러가 `raw 타입(Object)`로 인식하며, 중간에 `타입 형변환`이 추가된것을 확인할 수 있다.

![image](https://github.com/newcodingtest/-/assets/57785267/a7ceca7d-1693-4a02-9fa9-23db9a01e010)

<br>

**그러면 왜 raw 타입을 허용했을까??**

자바 이전 버전과의 하위 호환성 유지를 위해서

<br>

<br>

<br>

**List 의 안정성 예제**

```java
public class ListEx {
    public static void main(String[] args){
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));

        //런타임 환경에서 꺼낼때 에러가 난다.
        String exception = strings.get(0);
    }

    private static void unsafeAdd(List list, Object o){
        list.add(o);
    }
}
```

![image-20240612134420425](https://github.com/newcodingtest/-/assets/57785267/b2087780-ae89-4691-8fd4-a2c97e924611)

위 예제의 경우 unsafeAdd메서드는 List raw 타입을 입력받고있다. 메서드 호출시 이상없이 동작하지만 `strings.get(0)`으로 데이터를 가져올때 `cast 익셉션`이 터진다. -> raw 타입은 안정성을 보장할 수 없다.

<br>

직접 타입을 명시적으로 선언해서 안정성을 보장하자

```java
	..
	..
    private static void unsafeAdd(List<String> list, String o){
        list.add(o);
    }
    ..
```

![image-20240612140147673](https://github.com/newcodingtest/-/assets/57785267/673abf74-0af6-42b4-b43c-1c44351cf82d)

컴파일 단계에서 타입 안정성이 보장된다.

<br>

**Set의 안정성 예제**

```java
public class SetEx {
    static int numElementsInCommon(Set<?> s1, Set<?> s2){
        int result = 0;
        for (Object o1 : s1){
            if (s2.contains(o1)){
                result++;
            }
        }
        return result;
    }
    public static void main(String[] args){
        Set<Integer> set = new HashSet<>();

        Set mySet = set;
        mySet.add("test");

        System.out.println(Numbers.numElementsInCommon(Set.of(1,2,3), Set.of(1,2)));
    }
}
```

`Set mySet = set;`부분을 보면 raw 인 mySet이  Set<Interger> set 을 주입받고있다.

그러나 예상과는 다르게 Integer 가 아닌 String 을 추가할 수 있게 된다. `mySet.add("test");`

 <br>

이 또한 명시적으로 타입을 선언하자

```java
       ..
       Set<Integer> mySet = set;
       ..
```

컴파일 단계에서 검출된다.

![image-20240612141831593](https://github.com/newcodingtest/-/assets/57785267/0ab80f22-c239-4142-8485-721beba31773)


<br>

#### 예외케이스

**class, instanceof 를 제외**하고는 모두 타입을 선언하는게 권장된다.

```java
public class UseRawType<E> {

    private E e;

    public static void main(String[] args){
        System.out.println(UseRawType<String>.class);

        UseRawType<String> stringType = new UseRawType<>();

        System.out.println(stringType instanceof UseRawType<String>);
    }
}

```

1.**.class** 를 사용할땐 타입선언을 할 수 없다.



![image-20240612140147673](https://github.com/newcodingtest/-/assets/57785267/df1e9b5d-5286-4476-9b45-a752126aa071)

2.intanceof 사용시 타입선언은 의미가 없다.

 컴파일 단계에서 소거되므로 타입선언이 코드만 길어질뿐 의미가 없게된다.

![image-20240612142600881](https://github.com/newcodingtest/-/assets/57785267/c40a9b07-0939-48a8-afd5-93b8420de5bd)


<br>

<br>

때문에 아래처럼 타입선언을 하지 않고 사용해야 된다. 그 외에는 무조건 타입선언을 하자.

```java
System.out.println(UseRawType.class);

UseRawType<String> stringType = new UseRawType<>();

System.out.println(stringType instanceof UseRawType);
```


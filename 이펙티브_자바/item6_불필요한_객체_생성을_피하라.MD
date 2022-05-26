## 아이템 6_불필요한 객체 생성을 피하라

- 문자열

> 동일한 객체이기 때문에 매번 새로 만들필요가 없다.
>
> ```
> String test = "hello"; 
> String test1 = new String("hello"); //불 필요한 방법(동일한 문자열임에도 불구하고 강제로 객체공간을 만드는것)
> String test2 = "hello";
> 
> System.out.println(test == test1) //서로 다은 인스턴이 이기때문에 false
> System.out.println(test.equals(test1)) // true
> System.out.println(test.equals(test2)) // true 
> System.out.println(test == test2) // true
> ```
>
> JVM 에서 내부적으로 문자열은 Constant 상수 Pool에서 캐시중임
>
> 동일한 문자열은 같은 상수 pool에서 땡겨 사용함



- 정규 표현식, Pattern

> 만드는 비용이 비싸다, cpu 리소스를 사용하기 때문에

```
// 값비싼 객체를 재사용해 성능을 개선한다. (32쪽)
public class RomanNumerals {
    // 코드 6-1 성능을 훨씬 더 끌어올릴 수 있다!
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    // 코드 6-2 값비싼 객체를 재사용해 성능을 개선한다.
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }

    public static void main(String[] args) {
        boolean result = false;
        long start = System.nanoTime();
        for (int j = 0; j < 100; j++) {
            //TODO 성능 차이를 확인하려면 xxxSlow 메서드를 xxxFast 메서드로 바꿔 실행해보자.
            result = isRomanNumeralSlow("MCMLXXVI");
        }
        long end = System.nanoTime();
        System.out.println(end - start);
        System.out.println(result);
    }
}
```

> 동일한 패턴이 여러번 사용된다면 필드로 사용하자 (코드 6-1 -> 코드 6-2)



- 오토박싱

> 기본타입인 long과 참조타입인 Long을 비교하여 오토박싱 변환 과정에서 불 필요한객체가 생성될수 있다.

```
public class Sum {
    private static long sum() {
        // 시간이 보다 오래걸림
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }

    public static void main(String[] args) {
        long start = System.nanoTime();
        long x = sum();
        long end = System.nanoTime();
        System.out.println((end - start) / 1_000_000. + " ms.");
        System.out.println(x);
    }
}
```





#### **사용자제API**

**@Deprecated**

컴파일시 경고 메시지를 통해 사용 자제를 권장하는 API 라는 것을 클라이언트에 알려줄 수 있다,

**@deprecated**

문서화에 사용해, 왜 해당 API 사용을 지양하며, 그 대신 권장하는 API가 어떤 것인지 표기할 수 있다.

> 사용이 자제되길 원하는 메서드, 함수가 있으면 Deprecated 어노테이션을 사용하며
>
> 대체제가 있으면  @link로 문서화 하자

```
public class Deprecation {

    /**
     * @deprecated in favor of
     * {@link #Deprecation(String)}
     */
    @Deprecated(forRemoval = true, since = "1.2")
    public Deprecation() {
    }

    private String name;

    public Deprecation(String name) {
        this.name = name;
    }
}
```



#### 정규표현식 (https://regex101.com/)

> ​    name.split(","); 에서
>
> ​	split 같은 경우 한 단어에 대해서는 오히려 Pattern 보다 속도가 빠른 경우가 있다.

```
public class RegularExpression {

    private static final Pattern SPLIT_PATTERN = Pattern.compile(",");

    public static void main(String[] args) {
        long start = System.nanoTime();
        for (int j = 0; j < 10000; j++) {
            String name = "keesun,whiteship";
            name.split(",");
//            SPLIT_PATTERN.split(name);
        }
        System.out.println(System.nanoTime() - start);
    }
}
```



#### 가비지 컬렉션(https://developers.redhat.com/articles/2021/11/02/how-choose-best-java-garbage-collector#)



![img](https://mblogthumb-phinf.pstatic.net/MjAxNzA0MDFfNjIg/MDAxNDkxMDE5MTk2NTcw.WOdqWesEJyNQlOUvXIxlFThkwVyqPLj2Re5gP9E42tgg.n7CO3_kMib5r508vS84l4vMPEDqOCNZ3twrrQOgJx_og.PNG.kbh3983/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2017-04-01_%EC%98%A4%ED%9B%84_12.59.47.png?type=w800)

- Mark, Sweep, Compact

> Mark: 오브젝트가 참조를 갖고 있는지 없는지, 판단하여 없다면 사라지게 한다.
>
> Sweep: 필요없는 오브젝트 공간을 메모리 공간에서 없앤다.
>
> Compact: 파편화 된 메모리 공간을 압축하여 효율적인 공간을 만든다.

![img](https://mblogthumb-phinf.pstatic.net/MjAxNzAzMjZfMTA4/MDAxNDkwNTA4MTk1NzUy.czFSrJh0LAEEWTIVnhOBgIeDBwInCBaSvsejc6Ge234g.1QuznryvdmdF2kkj2I2zckkk7qlONFNoqvFePyuvvN8g.PNG.kbh3983/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2017-03-26_%EC%98%A4%ED%9B%84_3.02.50.png?type=w800)

- Young Generation(Eden, S0, S1), Old Generation, Permanent Generation

> 금방 죽는 오브젝트들은 Young Generation 으로 이동(Eden->S0->S1 순서로 이동) 
>
> Eden 은 객체가 생성되자 마자 저장되는곳 
>
> S0: Minor GC가 진행되고 살아남은 객체들이 저장되는곳
>
> S1: Minor GC가 발생하고 최종적으로 Eden과 S0에 살아 있는 객체들을 S1로 복사시킨다.
>
> 그 후 Eden과 S0을 clear 한다.
>
> 그리고 추후 Minor GC 발생시 비슷한 방법으로 Eden과 S1에 살아남은 객체들을 다시 S0으로 복사이동 시키고 clear 하면서 S0과 S1을 반복적으로 이동하며 최종적으로 살아남은 객체들을 Old영역으로 옮긴다.
>
> 다시말해 오래 사는 오브젝트들은 Old Generation 으로 이동
>
> Perm 영역은 프로그램 코드가 올라가는 부분, Old 영역에서 Full GC에서 살아남은 객체들이 Perm 영역으로 올라가지만 영원히 남아있지는 않는다. Perm 영역에서는 Major Gc가 일어나기 때문



- Minor GC, Full GC

> Young 영역에 GC가 발생한 경우 이를 Minor GC
>
> Old 영역에 GC가 발생한 경우 이를 Full GC

- Throughput, Latency(Stop-The-World), Footprint

> Throughput: 애플리케이션이 최대로 처리할수 있는 처리량(메모리가 될수도 있고, 소프트웨어/하드웨어 자원이 될수도..)
>
> Latency: 수많은 클라이언트들이 동시에 애플리케이션에 request 요청을 했을때 한번에 GC가 일어나면서 애플리케이션 전체가 멈추는 것



- 여러 GC종류들  Serial, **Parallel**(java8), CMS, **G1**(java11), **ZGC**, Shenandooh

  


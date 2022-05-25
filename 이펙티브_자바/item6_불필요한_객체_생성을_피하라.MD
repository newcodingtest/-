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


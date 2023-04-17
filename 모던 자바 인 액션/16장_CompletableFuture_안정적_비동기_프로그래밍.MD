#### Future를 이용하여 비동기를 구현하려고 할때 발생하는 문제점

```
public class Future비동기실행 {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<Double> future = executorService.submit(new Callable<Double>() {
            /**
             * 시간이 오래 걸리는 작업은 다른 스레드에서 비동기 적으로 실행
             * */
            @Override
            public Double call() throws Exception {
                
                /**
                 * 오래 걸리는 일
                 * */
                return doSomethingLongComputation();
            }
        });

        /**
         * 비동기 작업을 수행하는 동안 다른 작업 수행
         * */
        doSomething();

        /**
         * get 에서 스레드가 대기할 최대 타임아웃 시간을 설정하는것이 좋다.
         * 설정을 안하면 무한대기이므로 문제가 될 수 있음.
         * --> 결국 대기 해야된다는 것이 단점이 될 수 있음.
         * --> 때문에 자바 진영 에서는 CompletableFuture로 비동기 처리를 할 수 있음.
         * */
        try {
            Double result = future.get(1,TimeUnit.SECONDS);
        }catch (ExecutionException ee){

        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } catch (TimeoutException e) {
            throw new RuntimeException(e);
        }
    }
}
```
<br><br>

#### CompletableFuture를 이용한 비동기 메서드

```
    /**
     * 비동기로 가격을 계산하는 메서드
     * */
    public Future<Double> getPriceAsycn(String product){
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();

        new Thread(() -> {
            /**
             * 다른 스레드에서 비동기적으로 계산 수행
             * */
           double price = calculate(product);
           /**
            * 오래 걸리는 작업이 완료되면 Future에 값을 설정
            * */
           futurePrice.complete(price);
        }).start();
        
        /**
         * 계산 결과가 완료되길 기다리지 않고 Future 반환
         * */
        return futurePrice;
    }
```



```
 double price = calculate(product);
```

> 실제 가격을 계산할때 다른 스레드를 내부적으로 만들어서 처리하고, 오래 걸리는 계산 결과를 기다리지 않고 결과를 포함할 Future 인스턴스를 반환한다.

```
futurePrice.complete(price);
```

> 요청한 제품의 가격 정보가 도착하면 complete 메서드를 이용해서 CompletableFuture를 종료할 수 있다.


<br><br>
#### 비동기 API 사용

```
        ...
        ...
        long start = System.nanoTime();
        Future<Double> futurePrice = shop.getPriceAsycn("해피해킹 키보드");

          /**
          * 위의 futurePrice가 일하는동안 다른 일을 할수 있음(비동기)
          * */
        	doSomeThing();

            try{
                 /**
                  * 가격정보가 없으면 받을 때까지 블록한다.
                  * */
                double price = futurePrice.get();
                System.out.println("price is ", price);
            }catch(Exception e){
                throw new RuntimeException(e);
            }
        long end = System.nanoTime();
        System.out.println("process time ", (end-start)/100000);
```

```
			try{
                 /**
                  * 가격정보가 없으면 받을 때까지 블록한다.
                  * */
                double price = futurePrice.get();
                System.out.println("price is ", price);
            }catch(Exception e){
                throw new RuntimeException(e);
            }
```

> futurePrice.get()이 계산될때까지 기다린다(블록)

블록하지 않으면서 Future의 작업이 끝났을때만 이를 통지 받는 방법이 필요하다.

추후 알아보자


#### 에러 처리 방법

Future 에서는 에러 발생시 스레드에 영향을 미친다.

때문에 가격 계산시에 에러가 발생해도 계산은 계속 진행되기에 일의 순서가 꼬일수 있다. 

때문에 `future.get(1,TimeUnit.SECONDS);`과 같이 타임아웃을 `TimeoutException`으로 핸들링 할수 있지만

왜 에러가 발생했는지 알 수 있는 방법은 없다. 

때문에 `completeExceptionally 메서드를 이용해서 CompletableFuture 내부에서 발생한 예외를 클라이언트로 전달한다.`

##### **버전1**

```
    public static Future<Double> getPriceAsync(String product){
        CompletableFuture<Double> futurePrice = new CompletableFuture<>();
        new Thread(() -> {
                    try {
                        double price = calculate(product);
                        futurePrice.complete(price);
                    }catch (Exception ex){
                        /**
                         * 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종료
                         * */
                        futurePrice.completeExceptionally(ex);
                    }
        }).start();
        return futurePrice;
    }
```

##### **버전2**

버전1에 비해서 좀더 간결한 코드이다.

에러 핸들링 방법은 버전1과 같다.

```
Future<Double> getPriceAsync(String product){
		return CompletableFuture.supplyAsync(() -> calculate(product));
```



#### 비블록 코드 만들기

**상점을 검색하는 예제**

```
public class 비블록_코드만들기 {
    static List<Shop>shops = Arrays.asList(new Shop("hi"),
            new Shop("howru"),
            new Shop("imfine"),
            new Shop("andu"),
            new Shop("where"),
            new Shop("are"),
            new Shop("ugo"),
            new Shop("my"),
            new Shop("name"),
            new Shop("is"),
            new Shop("yoon"),
            new Shop("ju"),
            new Shop("young"),
            new Shop("howoldru"),
            new Shop("whichonedouwant")

    );
    static class Shop{
        private String name;
        public Shop(String name){
            this.name = name;
        }

        public String getName(){
            return this.name;
        }

        public float getPrice(String name){
            return name.length()+3;
        }
    }

    public static void main(String[] args) {
 		long start1 = System.nanoTime();
        findPrices1("howru");
        long end1 = System.nanoTime();
        System.out.println("순차:  \n"+(end1-start1) + " nsec");

        long start2= System.nanoTime();
        findPrices2("howru");
        long end2 = System.nanoTime();
        System.out.println("병렬스트림:  \n"+(end2-start2) + " nsec");

        long start3= System.nanoTime();
        findPrices3("howru");
        long end3 = System.nanoTime();
        System.out.println("CompletableFuture 비동기:  \n"+(end3-start3) + " nsec");
    }

    /**
     * 1. 모든 상점에 순차적으로 정보를 요청하는 메서드
     * */
   public static List<String> findPrices1(String product){
       return shops.stream()
               .map(shop -> String.format("%s price is %.2f", shop.getName(),shop.getPrice(product)))
               .collect(toList());
   }

    /**
     * 2. 병렬 스트림으로 정보를 요청하는 메서드
     * */
    public static List<String> findPrices2(String product){
        return shops.parallelStream()
                .map(shop -> String.format("%s price is %.2f", shop.getName(),shop.getPrice(product)))
                .collect(toList());
    }

    /**
     * 3. CompletableFuture로 비동기 요청하는 메서드
     * */
    public static List<String> findPrices3(String product){
        List<CompletableFuture<String>> priceFutures =
                shops.stream()
                        .map(shop -> CompletableFuture.supplyAsync(() ->
                                shop.getName() + " price is " + shop.getPrice(product)))
                        .collect(Collectors.toList());

        return priceFutures.stream()
                .map(CompletableFuture::join)
                .collect(toList());
    }
}
```

![image](https://user-images.githubusercontent.com/57785267/232446985-b9289511-fbd2-4158-a9f6-113e20c09ac4.png)

`1->2 방법으로 가면서 눈에 띄게 성능 개선이 되었다.`

```
    /**
     * 3. CompletableFuture로 비동기 요청하는 메서드
     * */
    public static List<String> findPrices3(String product){
        List<CompletableFuture<String>> priceFutures =
                shops.stream()
                        .map(shop -> CompletableFuture.supplyAsync(() ->
                                shop.getName() + " price is " + shop.getPrice(product)))
                        .collect(Collectors.toList());

        return priceFutures.stream()
                .map(CompletableFuture::join)
                .collect(toList());
    }
```

> 1. `CompletableFuture::join` 에서 join 메서드는 Future 인터페이스의 get 메서드와 같은 의미를 갖는다.
>
> 다만 `join은 아무 예외도 발생시키지 않는다는 점이 다르다.`
>
> 따라서 try catch 로 감쌀 필요가 없었다.

> 2. `map 연산을 2번 하는 이유를 스트림 연산은 게으른 특성이 있기 때문에 하나의 파이프라인으로 연산을 처리하면 모든 가격, 정보 요청 동작이 동기/순차적으로 이루어진다.`

![스트림의 게으름 때문에 순차 계산이 일어나는 이유와 이를 회피하는 법](https://leejaedoo.github.io/assets/img/steam_lazy.jpg)





`2,3번 버전 모두 내부적으로 Runtime.getRuntime().availableProcessors()가 반환하는 스레드 수를 사용하면서 비슷한 결과값이 나온다.`

그러나 CompletableFuture는 병렬 스트림 버전에 비해 각 작업에 이용할 수 있는 `다양한 Executor를 지정할 수 있는 장점이 있다.`



먼저, executor를 설정해주고

```
    private static final Executor executor =
            Executors.newFixedThreadPool(Math.min(shops.size(), 100),
                    new ThreadFactory() {
                        @Override
                        public Thread newThread(Runnable r) {
                            Thread t = new Thread(r);
                            /**
                             * 프로그램 종료를 방해하지 않는 데몬 스레드 생성
                             * */
                            t.setDaemon(true);
                            return t;
                        }
                    });
```

> 자바에서 일반 스레드가 실행 중이면 자바 프로그램은 종료되지 않는다. 
>
> 따라서 어떤 이벤트를 한없이 기다리면서 종료되지 않는 일반 스레드가 있으면 문제가 발생할 수 있다.
>
> 반면 데몬 스레드는 자바 프로그램이 종료될 때 강제로 실행이 종료될 수 있다. 
>
> 두 스레드의 성능은 같다.

요런식으로 파라미터에 executor를 사용한다고 선언하였다.

```
 .map(shop -> CompletableFuture.supplyAsync(() ->
                                        String.format("%s price is %.2f", shop.getName(),shop.getPrice(product)),executor))
```



`검색 상점이 증가하여도 일정 값까지(400개)는  CompletableFuture버전은 일정 성능을 유지할 수 있다.`

왜 그럴까? 아래 `스트림 병렬화 vs CompletableFuture 병렬화`부분을 참고해보자



#### 스트림 병렬화 vs CompletableFuture 병렬화

- I/O가  포함되지 않은 계산의 중심 동작을 실행할 땐 스트림 인터페이스가 가장 구현하기 간단하고 효율적일 수 있다.(모든 스레드가 계산 작업을 수행하는 상황에서는 프로세서 코어수 이상의 스레드를 가질 필요가 없다.)

- 작업이 I/O를 기다리는 작업을 병렬로 실행할 땐, CompletableFuture가 더 많은 유연성을 제공하며 대기/계산의 비율에 적잡한 스레드 수를 설정할 수 있다.

  

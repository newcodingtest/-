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

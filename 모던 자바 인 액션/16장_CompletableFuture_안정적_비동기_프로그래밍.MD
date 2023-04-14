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


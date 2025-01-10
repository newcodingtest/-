# ITEM 7. 다 쓴 객체 참조를 해제하라.

- 어떤 객체에 대한 래퍼런스가 남아있다면 해당 객체는 가비지 컬렉션의 대상이 되지 않는다.

- 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의하자(스택, 캐시, 리스너, 콜백)
- 참조 객체를 NULL 처리하는 일은 예외적인 경우이며 가장 좋은 방법은 유효 범위 밖으로 밀어내는 것이다.

<br>

**STACK 예제**

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    /* worst
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    */
    
    //best
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public static void main(String[] args) {
        Stack stack = new Stack();
        for (String arg : args)
            stack.push(arg);

        while (true)
            System.err.println(stack.pop());
    }
}
```

위의 pop 메서드에서 worst 케이스는 pop이 호출되면 인덱스만 감소할뿐 결국 push/pop을 반복해서 하게되면 메모리가 터질것이다.

때문에 pop된 객체는 명시적으로 null로 참조 해제를 하는것이 `첫 번째 방법`이다.

<br><br>

**캐시 예제**

```java
public class PostRepository {

    private Map<CacheKey, Post> cache;

    public PostRepository() {
        this.cache = new HashMap<>();
    }

    
    public Post getPostById(Integer id) {
        CacheKey key = new CacheKey(id);
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // TODO DB에서 읽어오거나 REST API를 통해 읽어올 수 있습니다.
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }
    
    public Map<CacheKey, Post> getCache() {
        return cache;
    }
}

public class CacheKey {

    private Integer value;

    private LocalDateTime created;

    public CacheKey(Integer value) {
        this.value = value;
        this.created = LocalDateTime.now();
    }
}


@Test
void cache() throws InterruptedException {
    PostRepository postRepository = new PostRepository();
    Integer key1 = 1
    postRepository.getPostById(key1);

    assertFalse(postRepository.getCache().isEmpty()); //false

    key1 = null;
    // TODO run gc
    System.out.println("run gc");
    System.gc();
    System.out.println("wait");
    Thread.sleep(3000L);

    assertTrue(postRepository.getCache().isEmpty()); //false
}
```

위에는 Integer를 캐시로 사용중인 예시이다.

key1 값을 명시적으로 null로 선언하고 gc를 실행해도 캐시는 비워지지 않는다.

그러나 PostRepository 에 있는 캐시 저장소인 HashMap 를 `WeakHashMap`을 사용하면  캐시가 비워지게 된다.

`WeakHashMap`은 key 값이 더이상 래퍼던스 되는 곳이 없다면 `GC`가 일어났을때 해당 엔트리를 `자동으로 제거`한다.

```java
   public Post getPostById(CacheKey key) {
        if (cache.containsKey(key)) {
            return cache.get(key);
        } else {
            // TODO DB에서 읽어오거나 REST API를 통해 읽어올 수 있습니다.
            Post post = new Post();
            cache.put(key, post);
            return post;
        }
    }	  


    @Test
    void cache() throws InterruptedException {
        PostRepository postRepository = new PostRepository();
        CacheKey key1 = new CacheKey(1);
        postRepository.getPostById(key1);

        assertFalse(postRepository.getCache().isEmpty()); //false
		
        // if you use this , the cache will be deleted
        //key1 = null;
        
        // TODO run gc
        System.out.println("run gc");
        System.gc();
        System.out.println("wait"); 
        Thread.sleep(3000L);

        assertTrue(postRepository.getCache().isEmpty()); ////false
    }
```

해당 경우에는 CacheKey의 라이프 범위를 getPostById 메서드 -> 테스트 코드로 더 확장시켰다. 이로서 캐시의 라이프사이클은 getPostById  메서드를 호출하고 끝나는것이 아닌 테스트 코드 종료까지는 캐시 래퍼런스가 살아있다. 때문에 캐시는 비워지지 않는다.

그러나 명시적으로 `key1 = null;` 하면 캐시는 지워진다.

이런식으로 참조를 해제하는 것이 `두 번째 방법`이다.

<br><br>

또는 Backgrount 쓰레드를 사용해서 주기적으로 삭제하는것이 `세 번째 방법`이다.

```java
    @Test
    void backgroundThread() throws InterruptedException {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
        PostRepository postRepository = new PostRepository();
        CacheKey key1 = new CacheKey(1);
        postRepository.getPostById(key1);

        Runnable removeOldCache = () -> {
            System.out.println("running removeOldCache task");
            Map<CacheKey, Post> cache = postRepository.getCache();
            Set<CacheKey> cacheKeys = cache.keySet();
            Optional<CacheKey> key = cacheKeys.stream().min(Comparator.comparing(CacheKey::getCreated));
            key.ifPresent((k) -> {
                System.out.println("removing " + k);
                cache.remove(k);
            });
        };

        System.out.println("The time is : " + new Date());

        executor.scheduleAtFixedRate(removeOldCache,
                1, 3, TimeUnit.SECONDS);

        Thread.sleep(20000L);

        executor.shutdown();
    }
```

그 외 `LRU(Least Recently Use) 캐시 자료구조`를 사용하는 것도 있다.

<br><br>



### WeekHashMap

Primitive 값들이 캐시의 key일 경우(`private Map<Integer, Post> cache;`) 아무리 명시적으로 null 처리를 하더라도 JVM 내부에 캐시형태로 저장되기 때문에 래퍼런스가 사라지지 않는다. 때문에 래퍼런스를 만들어서 사용것이 안전하다.(`private Map<CacheKey, Post> cache;`)

<br>

**strong 래퍼런스**

아래처럼 일반적으로 자료구조 사용하기 위해 생성자를 만드는 것이 strong 래퍼런스이다.

```java
    public ChatRoom() {
        this.users = new ArrayList<>();
    }
```

<br>

 **soft 래퍼런스**

gc를 한다고 해서 소프트 래퍼런스의 오브젝트가 없어지지 않는다.

 메모리가 정말로 필요할 때만 소프트 래퍼런스의 오브젝트가 없어진다.



 **week 래퍼런스**

gc가 일어나면 래퍼런스 오브젝트는 바로 없어진다.



 **pantom 래퍼런스**

아주 큰 오브젝트가 메모리에서 해제되는 시점을 알아야 하는 경우에만 사용

<br>

**자원을 반납하는 방법**

1. finalize
2. pantom 래퍼런스
3. try with resource

1은 최악의 방법, 2는 거의 사용안하는 방법, 3은 best 방법

<br>

`WeekReference` 개념은 사실 사용되기 극히 드믈기 때문에

일반적인 자료구조 사용해서 명시적으로 삭제하면서 관리하는것이 대게 더 좋다.



<br><br>









## 23-03-11
http완벽가이드 1~4장 

- 멱등이란?

- 멱등/비멱등 메소드 구분
- post, put, patch 메소드의 차이점
- http trace 메소드를  off 해야하는 이유
- http connect 메소드란?
- 캐시지원이 가능한 http 메소드는? 

- tcp 통신에서의 체크섬동작 원리

- 패킷 세그먼트 차이점

- 3 handshake 과정
- get과 post의 차이점은? get도 body를 사용할수 있는데 구지 post를?
- 인터넷에 구글을 검색하면 내부적으로 일어나는일(벡엔드관점)

<BR><BR><BR><BR><BR>
  ## 멱등이란?

> 어떤 대상에 같은 연산을 여러번 적용해도 결과가 달라지지 않는 성질`이다.

<BR><BR><BR>

## 멱등/비멱등 메서드 구분

![img](https://velog.velcdn.com/images%2Fneity16%2Fpost%2Fb8ad7ddf-0376-41ff-967a-18e3e79e5e13%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-02-08%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.55.56.png)

<BR><BR><BR>

## POST, PUT, PATCH 메소드의 차이점

> put만 멱등하고 나머진 멱등하지 않음

- POST

​				리소스의 생성을 담당

- PUT, PATCH

  - PUT은 리소스의 모든 속성을 수정할 때

  - PATCH는 리소스의 일부 속성을 수정할 때

    ex)put 을 patch처럼 사용하여 일부 속성만 수정하는 경우

    ​	수정 되지않은 나머지 속성값은 null 인식된다.

    ```
    // 기존 리소스
    {
        id: 1,
        name: 'evan',
        age: 30,
    }
    ```

    ```
    PUT users/1
    { age: 31 }
    ```

    ```
    // 새로운 리소스
    {
        id: 1,
        name: NULL,
        age: 31,
    }
    ```

  ### `PATCH`는 왜 멱등성이 보장되지 않는다는걸까?

  예를 들어, `PATCH` 메소드에 수정할 리소스의 일부분만 담아서 보내는 경우에는 당연히 멱등성이 보장된다.

  ```
  // 기존 리소스
  {
      id: 1,
      name: 'evan',
      age: 30,
  }
  PATCH users/1
  { age: 31 }
  // 새로운 리소스
  {
      id: 1,
      name: 'evan',
      age: 31, // 변경됨
  }
  ```

  이 `PATCH` 요청은 명확하게 `age`라는 필드를 `31`로 수정하는 행위만을 의미하므로 아무리 여러 번 수행해도 `age`는 늘 `31`이라는 값을 가지기 떄문이다.

  그럼 언제 `PATCH` 메소드가 멱등성 보장이 안될 수도 있다는 것일까?
  예를 들어

  ```
  PATCH users/1
  {
      $increase: 'age',
      value: 1,
  }
  ```

  이 요청의 `$increase` 필드의 값은 증가시키고 싶은 속성을 의미하고, `value` 필드의 값은 그 속성을 얼마나 증가시킬 것인지를 나타낸다.
  이 경우 API가 호출될 때마다 에반의 나이는 1씩 증가하기 때문에 이 API는 멱등성을 보장하지 않는다.

  이런식으로 `PATCH` 메소드를 사용하는 경우를 실제로 보지는 못했지만, 스펙 상 구현 방법에 대한 제약이 없기 떄문에 이런 식으로 사용한다고 해서 표준을 어기는 것은 아니라고 한다.
  따라서 `PATCH` 메소드는 API를 어떻게 구현하냐에 따라서 멱등성을 보장할 수도 있고 아닐 수도 있는 것이다.

  출처: https://oen-blog.tistory.com/211

  

  <BR><BR><BR>

### `PATCH`는 왜 멱등성이 보장되지 않는다는걸까?

예를 들어, `PATCH` 메소드에 수정할 리소스의 일부분만 담아서 보내는 경우에는 당연히 멱등성이 보장된다.

```
// 기존 리소스
{
    id: 1,
    name: 'evan',
    age: 30,
}
PATCH users/1
{ age: 31 }
// 새로운 리소스
{
    id: 1,
    name: 'evan',
    age: 31, // 변경됨
}
```

이 `PATCH` 요청은 명확하게 `age`라는 필드를 `31`로 수정하는 행위만을 의미하므로 아무리 여러 번 수행해도 `age`는 늘 `31`이라는 값을 가지기 떄문이다.

그럼 언제 `PATCH` 메소드가 멱등성 보장이 안될 수도 있다는 것일까?
예를 들어

```
PATCH users/1
{
    $increase: 'age',
    value: 1,
}
```

이 요청의 `$increase` 필드의 값은 증가시키고 싶은 속성을 의미하고, `value` 필드의 값은 그 속성을 얼마나 증가시킬 것인지를 나타낸다.
이 경우 API가 호출될 때마다 에반의 나이는 1씩 증가하기 때문에 이 API는 멱등성을 보장하지 않는다.

이런식으로 `PATCH` 메소드를 사용하는 경우를 실제로 보지는 못했지만, 스펙 상 구현 방법에 대한 제약이 없기 떄문에 이런 식으로 사용한다고 해서 표준을 어기는 것은 아니라고 한다.
따라서 `PATCH` 메소드는 API를 어떻게 구현하냐에 따라서 멱등성을 보장할 수도 있고 아닐 수도 있는 것이다.

출처: https://oen-blog.tistory.com/211

<BR><BR><BR>

## http trace 메소드를  off 해야하는 이유

https://devroach.tistory.com/117

spring에서의 대응방안

https://stackoverflow.com/questions/69824466/how-to-intercept-disable-http-trace-method-in-java-springboot

https://saltlee.tistory.com/214

<BR><BR><BR>

## HTTP connect 메서드란?

서버와 클라이언트 통신간 **proxy 서버를 통한 SSL tunneling에 주로 사용**

관련글 https://earth-95.tistory.com/m/138



## 캐시지원이 가능한 메서드는?

![img](https://velog.velcdn.com/images%2Fneity16%2Fpost%2Fb8ad7ddf-0376-41ff-967a-18e3e79e5e13%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202022-02-08%20%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB%2012.55.56.png)

> GET, HEAD, POST, PATCH



## tcp 통신에서의 체크섬동작 원리

## 패킷과 세그먼트의 차이

## 3way핸드쉐이크 과정

## GET과 POST의 차이점? GET도 body를 사용할 수 있는데 구지 POST를 사용할까?

## 인터넷에 구글을 검색하면 내부적으로 일어나는일


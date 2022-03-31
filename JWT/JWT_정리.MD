



## JWT (Json Web Token)

- #### 동작방식

![image](https://user-images.githubusercontent.com/57785267/160953222-91018388-e386-4c02-a9d8-1ebfa67e5cb8.png)

출처: https://devscb.tistory.com/m/80

```
4,5 번은 JWT 를 생성했던 서버와 같은 서버가 아니여도 된다.
ex)
MSA 방식과 같이 서비스 마다 여러 서버로 구성되있는 경우
인증서버 와 서비스를 제공하는 서버가 각각 다를 수 있음
```



- #### 구조

![image](https://user-images.githubusercontent.com/57785267/160953258-f921d46a-b133-47cc-a834-3d8cfd28a934.png)

사진출처: https://gorokke.tistory.com/181,https://brunch.co.kr/@jinyoungchoi95/1

> Header, Payload, Signature 3부분으로 이루어져 있음

> 각각의 구성 요소가 점(.)으로 구분되어 있는 형태이다.

- #### Header

2가지의 정보를 지니고 있음

```
{ 
 "alg": "HS256",
 "typ": JWT
}
```

alg: 해싱 알고리즘을 지정, 보통 HMAC SHA256 혹은 RSA 가 사용되며 이 해싱 알고리즘은 토큰을 검증 할때 사용되는 Signature 부분에서 사용됨

typ: 토큰의 타입을 지정

> 비밀키로 HS256 알고리즘이 적용되어 암호화가 되었음을 확인해볼수 있음



- #### PayLoad

여기에 담은 정보의 한 조각을 클레임이라고 부른다.

```
{
  "iss": "yjy",	
  "sub" : "1",
  "exp": "1485270000000",
  "https://gorokke.com/jwt_claims/is_admin": true, // 공개 클레임
  "userId": "11028373727102", // 비공개 클레임
  "username": "gorokke" // 비공개 클레임
}
```

iss: 토큰 발급자

sub: 토큰 제목

aud: 토큰 대상자

exp: 토큰 만료시간

nbf: 토큰 활성 날짜(이 날자 이전의 토큰은 활성화 되지 않음을 보장)

iat: 토큰 발급 시간

jti: JWT 토큰 식별자(iss 가 여러 명일 때 이를 구분하기 위한 값)



표준 스펙 이외에도 개발자가 추가해도 문제가 없다.

그러나 중요한 것은 p**ayload에 민감한 정보를 담지 않는 것이다**

payload는 특별한 암호화 없이 흔히 사용하는 base64 인코딩을 사용하기 때문에 서버가 아니더라고 그 값을 확인할 수가 있다.

```
암호화는 민감한 정보를 막아두어야 할때만 필요하다 그러나 이 자체만으로 많은 리소스를 사용하기 때문에 신중히 암호화를 적용해야 한다.
암호화 적용시 http 요청마다 한번의 복호화를 계속 해줘야 하기 때문이다.

그렇기 때문에 유출되었을때 큰 상관이 없는 비민감정보를 토큰에 담는것이 기본 스펙이 되는것이고
서버는 굳이 header나 payload를 암호화하지 않아도 되는 것이다.
```



- #### Signatue

```
- 인코딩 된 헤더와 페이로드를 점(.)으로 합친다.
- 헤더의 alg 속성에 명시된 알고리즘과 비밀키를 이용해 결과값을 해시한다.
- 해시된 결과를 Base64 URL로 인코딩 한다.
```

위의 단계애 따라 Signature가 만들어진다.

```
data = base64UrlEncode(header) + "." + base64UrlEncode(payload);
hash = RS256(data, private_key);
signature = base64UrlEncode(hash);
```

해당 서명이 만들어진것 을 확인할 수 있다.

```
POstGetfAytaZS82wHcjoTyoqhMyxXiWdR7Nn7A29DNSl0EiXLdwJ6xC6AfgZWF1bOsS_TuYI3OG85 -TRUNCATED- FfEbLxtF2pZS6YC1aSfLQxeNe8djT9YjpvRZA
```

출처:https://smoh.tistory.com/347



#### -토큰 검증

비밀키,공개키 정보: https://brunch.co.kr/@artiveloper/24 참고

![image](https://user-images.githubusercontent.com/57785267/160953321-9d7c2102-a289-4204-96ec-990fd17b9e2e.png)

출처: https://brunch.co.kr/@jinyoungchoi95/1

해당 사진은 Signature가 디코딩 되었을때의 사진이다.

정보를 보면 your-256-bit-secret 즉, 서버가 가지고 있는 개인키를 가지고 암호화 되어있는 상태이다.

따라서 Signature는 서버에 있는 비밀키로만 암호화를 풀 수 있으므로 다른 클라이언트는 임의로 Signature를 복호화 할 수 없다.

##### - 복호화 인증 과정

```
- JWT 토큰을 클라이언트가 서버로 요청과 동시에 전달
- 서버는 가지고 있는 개인키로 Signature를 복호화 한 후 Base64URLEncode(header)와 Base64URLEncode(payload)가 JWT의 것과 일치한지 확인후 인증을 허용한다.--서버가 기존에 발급했던 JWT와 비교를 한다. 
```





#### - 장점

> 토큰 자체에 사용자 인증에 필요한 모든 정보가 있기 때문에 별도의 인증 저장소가 필요없음

> 쿠키를 사용하지 않으므로 쿠키 취약점이 사라짐(클라 정보를 서버가 저장하지 않음)

> 서버에 부담이 적음



#### - 단점

> 1. 정보가 많아 질수록 토큰의 길이가 늘어남으로 네트워크에 부하를 줄수가 있음 -->
>
> 쿠키, 세션과는 다르게 base64 인코딩을 통해 정보 전달을 하므로

> 2. payload를 탈취하여 디코딩 하면 데이터를 볼수 있음 -> payload에는 비민감정보만 기입

> 3. stateless 하기 때문에 한번 만들어지면 제어가 불가 

2번의 경우 핵심 단점이다.

#### 토큰이 탈취당하면 만료될 때까지 대처 불가

3번의 이유때문에

#### 대처방안은?

JWT 유효시간을 짧게 가져가자

> ##### Sliding Session  방법

특정 서비스를 계속 사용하고 있는 특정 유저에게 만료시간을 연장

ex) 짧은 만료시간으로 인해 글을 쓰다 인증이 취소되어 글이 날아가는 불편한 경험을 한다면?

Sliding Session을 사용하면 글쓰기, 결제와 같은 특정 action을 유저가 행동했을때 새롭게 만료시간을 지정한 JWT를 제공하여 만료시간을 보완하는 방법

> **Refresh Token 방법**

가장 많이 사용하는 방법

처음 JWT 를 발급할 때 Access Token과 함께 Refresh Token을 같이 발급.

비교적 긴 시간(7일,30일)의 만료시간을 가진 Refresh Token을 발급하여 Refresh 보장

#### Refresh 토큰을 사용하면 세션의 단점 하나를 얻는것이다?

##### Refresh Token 동작 시나리오

```
1. 클라이언트가 ID,PW로 서버에게 인증을 요청, 서버는 이를 확인하여 Access Token과 Refresh Token을 발급
2. 클아이언트는 Refresh Token을 본인이 잘 저장하고 Access Token으로 서버에게 자유롭게 요청
3. 서비스를 이용중 Access Token이 만료되어 사용할수 없다는 오류를 서버로 부터 전달 받음
4. 클라이언트는 Access Token의 만료 사실을 인지하고 본인이 갖고있던 Refresh Token으로 서버에게 전달하여 새로운 Access Token의 발급을 요청
5. 서버는 Refresh Token을 받아 서버의 Refresh Token Storage에 토큰의 존재 여부를 확인후 있다면 새로운 Access Token을 생성하여 전달
6. 이후 2번으로 돌아가서 동일한 작업 진행
```

5번에 등작한 Refresh Token Storage는 세션 저장소와 똑같은 역할을 한다

세션 방식과 차이 없이 특정 Storage에 I/O 작업이 발생하게 된다. 이로인해 세션의 단점을 하나 가져 가는 것이다.

단, 이를 사용함으로써 세션처럼 토큰 자체가 탈취되었다고 판단하면 Refresh Token Storage를 초기화하여 탈취된 토큰이 더 Refresh 못하도록 막는 옵션이 생긴다.

#### **그러나 세션과 다른점은?**

세션을 항상 인증 요청할 때마다 세션ID를 세션 저장소와 비교를 해야하므로

1분당 천번의 요청이 있을땐 I/O 도 천번 작동한다.

그러나

Refresh Token은 Access Token이 만료 되었을때만 I/O작업이 작동되기 때문에

세션과는 다르다고 말할 수 있다.

#### 

출처:https://brunch.co.kr/@jinyoungchoi95/1

https://brunch.co.kr/@jinyoungchoi95/1

https://gorokke.tistory.com/181

https://smoh.tistory.com/347

https://devscb.tistory.com/m/80

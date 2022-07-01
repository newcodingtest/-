### Property 읽는 과정

**Property**를 바인딩하는 다양한 방법이 존재하듯이 바인딩할 **Property**파일을 **찾는 방법 또한 순서**가 존재합니다.

Spring Boot 동작 시 **Property**를 찾는 순서입니다. (몇 가지는 제외하였습니다.)            

 

| **1. Terminal에서 명령어 입력**                        | --spring.properties.active=real 등                      |
| ------------------------------------------------------ | ------------------------------------------------------- |
| **2. Java 시스템 속성**                                | System.getProperties()                                  |
| **3. jar 파일과 같은 경로에 존재하는 properties 파일** | .properties, .yml profile 모두 적용 가능                |
| **4. jar 파일과 함께 패키징된 properties 파일**        | src/main/resources 경로에 위치한 application.properties |

<br><br><br>



#### 나는 첫번째 방법을 사용

터미널에서 옵션으로 설정해줄수 있다.

> java -jar 실핼할파일.jar --spring.profiles.active=외부프로퍼티이름



아래 위치처럼 같은 jar 파일 내에 프로퍼티가 있으면 가능하다.

![image](https://user-images.githubusercontent.com/57785267/176814040-89b2272e-284f-4c7a-aecc-909fb1d2f0e4.png)

<br><br><br>


## Spring 프로퍼티 우선순위

1. [spring-boot-devtools](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools)를 활성화 시켰을 때 `$HOME/.config/spring-boot` 디렉토리에 안에서 제공하는 프로퍼티
2. 테스트에 사용한 `@TestPropertySource`가 제공하는 프로퍼티
3. `@SpringBootTest` 또는 슬라이스 테스트용 애노테이션의 `properties` 속성으로 제공하는 프로퍼티
4. 커맨드 라인 아규먼트
5. `SPRING_APPLICATION_JSON` 환경 변수 또는 시스템 프로퍼티에 인라인 JSON으로 정의되어 있는 프로퍼티
6. ServletConfig 초기 매개변수
7. ServletContext 초기 매개변수
8. java:comp/env에 들어있는 JNDI 애트리뷰트
9. 자바 시스템 프로퍼티 (System.getProperties())
10. 운영체제 환경 변수
11. `RandomValuePropertySource`. `random` 접두어를 가지고 있는 프로퍼티, `random.*` 에 무작위 값을 제공하는 프로퍼티 소스.
12. JAR 패키지 외부에 있는 특정 프로파일용 애플리케이션 프로퍼티. (application-{profile}.properties 또는 YAML
13. JAR 패키지 내부에 있는 특정 프로파일용 애플리케이션 프로퍼티. (application-{profile}.properties 또는 YAML
14. JAR 패키지 외부에 있는 애플리케이션 프로퍼티. (application.properteis 또는 YAML)
15. JAR 패키지 내부에 있는 애플리케이션 프로퍼티. (application.properteis 또는 YAML)
16. `@Configuration` 클래스에 사용한 `@PropertySource`로 읽어들인 프로퍼티
17. SpringApplication.setDefaultProperties()로 설정할 수 있는 기본 프로퍼티

출처: https://www.whiteship.me/spring-boot-external-config/


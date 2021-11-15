
- MSA와 같은 분산 환경은 서비스 간의 원격 호출로 구성이 된다. 원격 서비스 호출은 IP 주소와 포트를 이용하는 방식
- 클라우드 환경이 되면서 서비스가 오토 스케일링등에 의해서 동적으로 생성되거나 컨테이너 기반의 배포로 인해서, 서비스의 IP가 동적으로 변경되는 일이 잦아졌다.
- 서비스 클라이언트가 서비스를 호출할때 서비스의 위치 (즉 IP주소와 포트)를 알아낼 수 있는 기능을 Service Discovery라 한다.

## Spring Cloud Netflix Eureka

- Spring Cloud에서 제공하는 Service Discovery 서비스
- 각각의 마이크로 서비스의 어디에 누가 저장되어 있으며 요청정보에 따라 필요한 서비스의 위치를 알려주는 역할

## Eureka Service Discovery - 프로젝트 생성

- 프로젝트 구조
    
    ![Untitled](https://user-images.githubusercontent.com/72686708/141733221-2b08f062-e917-4d25-ad0c-b69df3f56885.png)
    
- dependency
    
    ![Untitled 1](https://user-images.githubusercontent.com/72686708/141733235-321da2a9-3798-403e-814d-5c880dea5139.png)
    
- application.yml
    
    ```yaml
    server:
      port: 8786
    
    spring:
      application:
        name: discoveryservice
    
    eureka:
      client:
        register-with-eureka: false
        fetch-registry: false
    ```
    
    - `rigister-with-eureka`, `fetch-registry`
        - 유레카 서버에 자기 자신의 정보를 등록하지 않음
- Applicaiotn.java
    
    ```java
    @SpringBootApplication
    @EnableEurekaServer
    public class DiscoveryserviceApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(DiscoveryserviceApplication.class, args);
        }
    
    }
    ```
    
    - `@EnableEurekaServer`
        - 해당 웹 서비스를 유레카 서버로 설정

## User Service - 프로젝트 생성

- **프로젝트 구조**
    
    ![Untitled 2](https://user-images.githubusercontent.com/72686708/141733254-4c9389b9-4526-40eb-bdc5-2c69012e5520.png)
    
- **dependency**
    - Spring Boot Devtools
    - Lombok
    - Spring Web
    - Eureka Discovery Client
- **Application.java**
    
    ```java
    @SpringBootApplication
    @EnableEurekaClient
    public class UserServiceApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(UserServiceApplication.class, args);
        }
    
    }
    ```
    
    - `@EnableEurekaClient`
        - 해당 웹서비스를 유레카 클라이언트로 설정
- **application.yml**
    
    ```yaml
    server:
      port: 9001
    
    spring:
      application:
        name: user-service
    
    eureka:
      client:
        register-with-eureka: true
        fetch-registry: true
        service-url:
          defaultZone: http://127.0.0.1:8786//eureka
    ```
    
    - `register-with-eureka`
        - 유레카의 레지스트에 해당 서버를 등록할지 여부
    - `fetch-registry`
        - 유레카 서버로부터 인스턴스들의 정보를 주기적으로 가져올 것인지를 설정하는 속성
    - `service-url`
        - `defaultZone`: 유레카 엔드포인트에 작성한 마이크로 서비스 정보를 등록
- **서버 기동**
    - 유레카 디스커버리 서버가 동작되어 있는 상태에서 서버를 기동해야한다.

## User Service 등록

### User Service 인스턴스 추가

UserService의 역할을 하는 인스턴스를 두개 설정하여 유레카 서버에 등록해보자

![Untitled 3](https://user-images.githubusercontent.com/72686708/141733272-a04e4261-d50c-4900-9d8e-7230000e1d4a.png)

1. 인텔리제이 위측의 서버 셀렉트바 클릭
2. Edit Configuration
3. copy 버튼 클릭 (UserService 복사)

### 포트 번호 변경

![Untitled 4](https://user-images.githubusercontent.com/72686708/141733285-c8e9f1a1-f122-47f2-9fdc-c47e565118cb.png)

- VM Options: `-Dserver.port=9002`

### mvn run 사용

![Untitled 5](https://user-images.githubusercontent.com/72686708/141733295-7191030c-dd1d-4d2a-b7f3-4eabefc8087f.png)

**유레카 서버 확인**

![Untitled 6](https://user-images.githubusercontent.com/72686708/141733308-ee4fffc8-277b-45d6-b8e8-28c47e423cc2.png)

- 2개의 USER-SERVICE 인스턴스가 등록된 것을 확인 가능하다.

## 여러개의 Instance 기동

### 랜덤 포트 사용

- application.yml 파일의 서버 포트를 0 번으로 설정 (랜덤포트)
    
    ```yaml
    server:
      port: 0
    ```
    
- mvn spring-boot run을 사용하여 랜덤포트로 서버를 기동
- application.yml 파일의 인스턴스 아이디 설정
    
    ```yaml
    eureka:	
      instance:
        instance-id: ${spring.cloud.client.hostname}:${spring.application.instance_id:${random.value}}
    ```
    

**유레카 서버 확인**

![Untitled 7](https://user-images.githubusercontent.com/72686708/141733325-e81554b5-8f9c-4d00-8c8a-646cca27e2fa.png)

## 참고 - 인텔리제이 프로젝트 여러개 열기

- [링크](https://wakestand.tistory.com/632?category=768176)

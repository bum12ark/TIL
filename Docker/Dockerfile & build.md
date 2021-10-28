## Docker file

- 도커에서 이미지를 만드는 방법은 크게 두가지가 존재한다.
    1. 이미 생성된 이미지를 run 하여 만든 컨테이너를 commit 명령어를 통해 이미지로 만들 수 있다.
    2. 만들고 싶은 이미지 형식에 맞게 도커 파일을 작성하여 build 명령어를 통해 이미지를 만들 수 있다.
- 예
    
    ```docker
    FROM ubuntu:20.04
    RUN apt update && apt install -y python3
    WORKDIR /var/www/html
    COPY ["index.html", '."]
    EXPOSE 8000
    ENTRYPOINT ["python3", "-u", "-m", "http.server"]
    ```
    

### commit vs build

- 공통점
    - 이미지를 만드는 명렁어
- 차이점
    - **commit**
        - 이미 사용하고 있는 컨테이너가 있을 때 그 컨테이너를 이미지로 만드는 행위
        - 백업과 같은 느낌으로 사용
    - **build**
        - dockerfile을 통해서 사용자가 만들고 싶은 이미지를 구체적으로 기록하여 만드는 이미지 생성하는 행위

## 예제

- 웹서버 이미지를 만들어 해당 이미지를 run하여 사용하고 싶다.

### commit 사용

```docker
# ubuntu:20.04 이미지를 통해 web-server 라는 이름으로 컨테이너를 생성
docker run --name web-server -it ubuntu:20.04
```

- `docker run -it` : 해당 컨테이너의 쉘을 실행

```docker
# web-server 컨테이너를 통해 web-server-commit 이미지를 생성
docker commit web-server web-server-commit

docker images
```

- `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
- `docker images`
    
    ![Untitled](https://user-images.githubusercontent.com/72686708/139217751-5d427da3-1566-4fe4-aad2-9f5db96553f9.png)
    

### Dockerfild: build 사용

- `docker build [OPTIONS] PATH | URL | -`
- Dockerfile
    
    ```docker
    FROM ubuntu:20.04
    ```
    
- CLI
    
    ```docker
    docker build -t web-server-build .
    ```
    
- GUI 확인
    
    ![Untitled 1](https://user-images.githubusercontent.com/72686708/139217775-1011c2c1-f53b-4a1b-b100-f5fd89a144b9.png)
    

### 예제 - 스프링 부트 프로젝트로 이미지 만들기

1. 스프링 부트 프로젝트 생성
    - build.gradle
        
        ```
        plugins {
            id 'org.springframework.boot' version '2.5.6'
            id 'io.spring.dependency-management' version '1.0.11.RELEASE'
            id 'java'
        }
        
        group = 'hello.docker'
        version = '0.0.1-SNAPSHOT'
        sourceCompatibility = '11'
        
        repositories {
            mavenCentral()
        }
        
        dependencies {
            implementation 'org.springframework.boot:spring-boot-starter-web'
            testImplementation 'org.springframework.boot:spring-boot-starter-test'
        }
        
        test {
            useJUnitPlatform()
        }
        ```
        
        - spring-boot-stater-web 의존성만 추가하여 기본적인 테스트 프로젝트 생성
    - 컨트롤러
        
        ```java
        @RestController
        public class HelloController {
            
            @RequestMapping("hello-docker")
            public String helloDocker() {
                return "Hello Docker!";
            }
        }
        ```
        
2. gradle build를 통해 jar 파일을 생성
    - build/libs 폴더에 jar파일이 생성된 것을 확인 할 수 있다.
        
        ![Untitled 2](https://user-images.githubusercontent.com/72686708/139217814-f49a7528-8029-4520-a87e-b0964e79c248.png)
        
    - `java -jar build/libs/docker-demo-0.0.1-SNAPSHOT.jar` 명령어를 통해 정상 동작하는지 확인 가능하다.
        
        ![Untitled 3](https://user-images.githubusercontent.com/72686708/139217841-2c9e2351-f55e-4eb7-9eec-5a69d054f50c.png)
        
        - 스프링 부트가 로컬에서 정상적으로 동작하는 것을 확인
3. SpringBoot 웹 프로젝트 jar로 도커파일 만들기
    
    ```docker
    FROM openjdk:11
    ADD build/libs/docker-demo-0.0.1-SNAPSHOT.jar app.jar
    ENV JAVA_OPTS=""
    ENTRYPOINT ["java", "-jar", "/app.jar"]
    ```
    
    - FROM: 베이스 이미지를 지정
    - ADD: 현재 경로의 JAR 파일을 복사하여 app.jar 파일로 만들어 이미지에 적재
    - ENV: 환경 변수
    - ENTRYPOINT: 이미지가 동작하면서 실행시킬 명렁어 입력, 이미지가 RUN 될때 java -jar /app.jar를 실행
4. 도커 파일을 통해 이미지 생성
    
    ```docker
    docker build --tag docker-demo-build .
    ```
    
    - docker images 명렁어를 통해 성공정으로 이미지가 생성된 것을 확인할 수 있다.
        
        ![Untitled 4](https://user-images.githubusercontent.com/72686708/139217859-c9ef863c-ee9a-43e7-b3f7-36d5f3c25a52.png)
        
5. 생성한 docker 이미지를 실행하여 동작 확인
    
    ```docker
    docker run --name ws-springweb -p 8081:8080 docker-demo-build
    ```
    
    - `-p 8081:8080`
        - 8081 포트로 접근 가능하며 톰캣의 8080포트로 포트 포워딩 시켜준다
6. 동작 확인
    
    ![Untitled 5](https://user-images.githubusercontent.com/72686708/139217881-4db7fb54-f5fc-4c61-9586-c23be540f560.png)
    
    ![Untitled 6](https://user-images.githubusercontent.com/72686708/139217901-57f5f850-148c-4eca-a270-ecd46920fbf4.png)

# Mircroservice와 Spring Cloud의 소개

생성일: 2021년 11월 14일 오전 11:53

## 소프트웨어 아키텍처

### The History of IT System

- 1960 ~ 1980s : Fragile, Cowboys
    - Mainframe, Hardware
- 1990 ~ 2000s : Robust, Distributed
    - Changes
- 2010s ~ : Resilient/Anti-Fragile, Cloud Native
    - Flow of value의 지속적인 개선

### Antifragile의 특징

- **Auto scaling**
    - 자동 확장성
    - 시스템의 인스턴스를 하나의 오토 스케일링 그룹으로 지정
    - 해당 그룹의 최소 인스턴스 개수를 지정하여 사용량에 따라 자동으로 인스턴스를 증가할 수 있는 환경
- **Microservices**
    - 클라우드 네이티브 아키텍처의 핵심
    - 전체 서비스를 구축하고 있는 개별적인 모듈을 독립적으로 개발, 배포, 운영하도록 세부화된 서비스
- **Chaos engineering**
    - 시스템이 격동의 예측치 못한 상황을 견딜 수 있도록 신뢰성을 쌓기 위해 운영 중인 소프트웨어 시스템에 실험을 하는 규율
- **Continous deployments**
    - 자동화 된 배포를 통해 소프트웨어 기능이 자주 제공되는 소프트웨어 엔지니어링 접근 방식

## Cloud Native Architecture

- **확장 가능한 아키텍처**
    - 시스템의 수평적 확장에 유연
    - 확장된 서버로 시스템의 부하 분산, 가용성 보장
    - 시스템 또는, 서비스 애플리케이션 단위의 패키지 (컨테이너 기반 패키지)
    - 모니터링
- **탄력적 아키텍처**
    - 서비스 생성 - 통합 - 배포, 비즈니스 환경 변화에 대응 시간 단축
    - 분할된 서비스 구조
    - 무상태 통신 프로토콜
    - 서비스의 추가와 삭제 자동으로 감지
    - 변경된 서비스 요청에 따라 사용자 요청 처리 (동적 처리)
- **장애 격리 (Fault isolation)**
    - 특정 서비스에 오류가 발생해도 다른 서비스에 영향 주지 않음

## Cloud Native Application

- **Microservices**
- **CI/CD**
- **DevOps**
- **Containers**

### CI/CD

- 지속적인 통합, CI
    - 통합 서버, 소스 관리 (SCM), 빌드 도구, 테스트 도구
    - ex) Jenkins, Team CI
- 지속적 배포
    - Continuous Delivery
    - Continuous Deployment
    - Pipe line
- 카나리 배포와 블루그린 배포

### DevOps

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled.png)

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled%201.png)

- 소프트웨어의 개발과 운영의 합성어
- 소프트웨어 개발자와 정보기술 전문가 간의 소통, 협업 및 통합을 강조하는 개발 환경이나 문화

### Container 가상화

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled%202.png)

- OS 레벨에서 어플리케이션 실행 환경을 격리함으로써 마치 다른 OS에서 동작하는 것과 같은 가상 실행 환경을 제공하는 기술
- 컨테이너는 VM과 유사하지만 격리 속성을 완화하여 애플리케이션 간에 운영체제(OS)를 공유하므로 각각의 컨테이너는 보다 가벼움

## 12 Factors

**특징**

- SaaS 앱을 개발하기 위한 방법론
- 설정 자동화를 위한 절차 체계화로 새로운 개발자가 프로젝트에 참여하는데 드는 시간과 비용을 최소화

**12개의 요소**

1. 코드베이스
    - 버전 관리되는 하나의 코드베이스와 다양한 배포
2. 종속성
    - 명시적으로 선언되고 분리된 종속성
3. 설정
    - 환경에 저장된 설정
4. 백엔드 서비스
    - 백엔드 서비스를 연결된 리소스로 취급
5. 빌드, 릴리즈, 실행
    - 철저하게 분리된 빌드와 실행 단계
6. 프로세스
    - 애플리케이션을 하나 혹은 여러개의 무상태 프로세스로 실행
7. 포트 바인딩
    - 포트 바인딩을 사용해서 서비스를 공개함
8. 동시성
    - 프로세스 모델을 사용한 확장
9. 폐기 가능
    - 빠른 시작과 그레이스풀 셧다운을 통한 안정성 극대화
10. 개발/프로덕션 환경 일치
    - 개발, 스테이징, 프로덕션 환경을 최대한 비슷하게 유지
11. 로그
    - 로그를 이벤트 스트림으로 취급
12. Admin 프로세스
    - admin/maintenace 작업을 일화성 프로세스로 실행

## Monolithic vs. Microservice

### Monolith Architecture

- 모든 업무 로직이 하나의 애플리케이션 형태로 패키지 되어 서비스
- 애플리케이션에서 사용하는 데이터가 한곳에 모여 참조되어 서비스되는 형태

### Microservice

> Small autonomous services that work together
(함께 작동하는 작은 규모의 서비스들)
- Sam Newman
> 
- HTTP 통신을 이용한 작은 규모의 여러 서비스들의 묶음
- 비즈니스 기능을 중심으로 구축되며 완전하게 자동화면 배포 시스템을 사용해야 한다.
- 각각의 서비스들은 최소한의 중앙 집중식 관리가 되어야 하며, 서로 다른 프로그래밍 언어와 데이터 저장 기술을 사용할 수 있다.

## Microservice Arhitecture란?

### Microservice의 특징

1. Challenges
2. Small Well Cosen Deplyable Units
3. Bounded Context
4. RESTful
5. Configuration Management
6. Cloud Enabled
7. Dynamic Scale Up And Scale Down
8. CI/CD
9. Visibility

## SOA vs. MSA

- 서비스의 공유 지향점
    - SOA : 재사용을 통한 비용 절감
    - MSA : 서비스 간의 결합도를 낮추어 능동적으로 대응
- 기술 방식
    - SOA : 공통의 서비스를 ESB에 모아 사업 측면에서 공통 서비스 형식으로 서비스 제공
    - MSA : 각 독립된 서비스가 노출된 REST API 사용

### RESTful Web Service

> A way  to grade your API according to the constraints of REST
(REST의 제약 조건에 따라 API를 평가하는 방법)
- Leonard Richardson
> 

**REST 성숙도 모델**

- LEVEL 0
    - rest 스타일로 soap 웹 서비스 노출
    - http://server/getPosts
    - http://server/deletePosts
    - http://servier/doThis
- LEVEL 1
    - 적절한 URI를 사용하여 리소스 노출
    - http://server/accounts
    - http://server/acounts/10
    - 노트: http 메서드의 부적절한 사용
- LEVEL2
    - Level1 + HTTP Methods
- LEVEL3
    - Level2 + HATEOAS
    - DATA + NEXT POSSIBLE ACTIONS

**고려해야할 점**

- **Consumer first**
    - 개발자 중심의 설계 방식 보다는 소비자(이 API를 쓰는 다른 개발자) 입장에서 설계해야 한다.
- **Make best use of HTTP**
- **Request Methods**
    - GET, POST, PUT, DELETE
- **Response Status**
    - 200, 404, 400, 201, 401
- **No secure info in URI**
- **Use plurals**
    - 복수형태로 쓰는 것이 일반적이다
    - prefer /users to /user
    - prefer /users/1 to /user/1
- **User nouns for resources**
    - 모든 리소스는 가능하면 명사형태로 표시
- **For exceptions**
    - define a consistent approach
        
        ```
        /search
        PUT /gists/{id}/star
        DELETE /gists/{id}/star
        ```
        

## Microservice Arichitecture Structures

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled%203.png)

### Service Mesh

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled%204.png)

- MSA 인프라 → **미들웨어**
    - 프록시 역할, 인증, 권환 부여, 암호화, 서비스 검색, 요청 라우팅, 로드 밸런싱
    - **자가 치유 복구 서비스**
- 서비스간의 통신과 관련된 기능을 자동화

### MSA 표준 구성요소

- CNCF (Cloud Native Computing Foundation)
    - Cloud Native Interactive Landscape)
    - [https://landscape.cncf.io/](https://landscape.cncf.io/)
- Cloud Native를 구축하는데 있어서 서로 상호 연관될 수 있는 서비스들을 제공한다.

### MSA 기반 기술

![Untitled](Mircroservice%E1%84%8B%E1%85%AA%20Spring%20Cloud%E1%84%8B%E1%85%B4%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20e092cc6eac544501b10a113ac877aea9/Untitled%205.png)

## Spring Cloud란?

- **Spring Boot + Spring Cloud**
    - 자신이 사용하는 스프링 부트의 버전에 따라 특정 라이브러리를 사용하지 못하거나 방법이 바뀌기 때문에 버전에 유의하여 사용해야한다.
- **Main Projects**
    - Spring Cloud Config
    - Spring Cloud netflix
    - Spring Cloud Security
    - Spring Cloud Sleuth
    - Spring Cloud Starters
    - Spring Cloud Gateway
    - Spring Cloud OpenFeign
    - ...

### Spring Cloud 서비스

- 환경 설정 관리
    - **Spring Cloud Config Server**
    - 다양한 마이크로 서비스에서 사용할 수 있는 정보를 한곳에서 설정 가능!
- 서비스의 등록과 위치정보 확인
    - **Naming Server (Eureka)**
- 로드 밸런싱
    - **Ribbon (Client Side)**
    - **Spring Cloud Gateway**
- 사용하기 쉬운 REST 클라이언트
    - **FeignClient**
- 시각화와 모니터링
    - **Zipkin Distributed Tracing**
    - **Netflix API gateway**
- 장애 복구 패턴
    - **Hystrix**
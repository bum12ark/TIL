
## 애플리케이션 개요

![Untitled](https://user-images.githubusercontent.com/72686708/142635879-61abe331-4431-4275-87ac-a42491160075.png)

- CATALOG-SERVICE
    - 상품
- USER-SERVICE
    - 회원
- ORDER-SERVICE
    - 주문

## 애플리케이션 구성

![Untitled 1](https://user-images.githubusercontent.com/72686708/142635899-a4ebbe4f-c947-4612-917a-aaa238cb22b5.png)

![Untitled 2](https://user-images.githubusercontent.com/72686708/142635910-f962e765-aefd-4df6-bbb5-b1b6fabbc338.png)

- **유레카 서버**
    - Registery service (Service Discovery)
    - 마이크로 서비스들의 등록, 위치 정보 등을 가지고 있음
    - 마이크로 서비스들과의 주기적인 핑을 통해 해당 서버의 기동유무 또한 파악
- **Spring Cloud Gateway**
    - API Gateway
    - 전체 시스템에 대한 전, 후처리(예. 로깅, 인증)를 쉽게 할 수 있음
    - 로드 밸런싱 (부하 분산), 서비스 라우팅 역할
- **Config Server**
    - 모든 서비스의 구성 정보를 담고 있는 서버
    - 각 서비스의 application 파일에서 관리하는 것이 아닌 하나의 서버에서 관리
    - 구성 정보가 바뀔 때 마다 해당 서버를 재기동하지 않아도 된다는 장점 존재
- **Kafka**
    - 메세징 큐 서비스
    - 각 마이크로 서비스의 데이터 베이스 동기화를 위해 사용

## 애플리케이션 구성요소

![Untitled 3](https://user-images.githubusercontent.com/72686708/142635929-ba6180a9-1b42-4aab-b140-3ebfb7c7dca4.png)

## 애플리케이션 API 목록

![Untitled 4](https://user-images.githubusercontent.com/72686708/142635947-aab9186c-6159-4add-b311-30aef7d953e9.png)

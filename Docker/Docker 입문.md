## 소개

### Virtual Macines 와 Docker

![https://blog.kakaocdn.net/dn/dJN045/btqGbOTevZ7/dok3jbKoIv8wwZbSJ2Nf9k/img.jpg](https://blog.kakaocdn.net/dn/dJN045/btqGbOTevZ7/dok3jbKoIv8wwZbSJ2Nf9k/img.jpg)

- **Virtual Macines**
    - 컴퓨터의 리소스를 분할하여 사용
    - 속도가 느리고 주변 장치와의 완벽한 호환이 어려운 단점 존재
- **Docker**
    - 도커 컨테이너는 VM을 사용하지 않고 도커 엔진을 사용하여 동작
    - 도커 엔진 위의 컨테이너에 이미지라는 실행에 필요한 파일과 설정값들을 담아 사용
    - 이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있으므로 추가적인 설치가 불필요
    - OS를 별도로 설치하지 않기 때문에 실행 속도 측면(성능)에서 유리하다.

## 설치

- 도커와 같은 컨테이너 기술은 **Linux 운영체제** 기술
- 윈도우, 맥과 같은 운영체제의 경우 도커가 가상 머신 위에 리눅스 운영체제를 만들어 실행 가능토록 해준다.
1. [Docker 홈페이지 접속](https://www.docker.com/)
2. 상단 메뉴의 Developers → Docs 클릭
3. Download and install
4. 운영체제에 맞는 Docker 설치
5. 명령 프롬포트 실행  `docker images` 명령어 실행 시 에러가 나지 않는다면 정상 설치된 것

## 이미지 pull

### 용어 정리

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38215111-179f-4378-80b0-1c5c8ef86a93/Untitled.png)

- **docker hub**
    - docker를 사용하는데 필요한 이미지를 다운로드 하게 지원해주는 레지스트리 (ex. App Store)
- **image**
    - 도커 허브에서 다운로드 해서 가지고 있는 것을 이미지라 한다. (ex. Program)
    - 하나의 프로그램이 여러 프로세스를 가질 수 있는 것 처럼 이미지도 여러 컨테이너를 가질 수 있다.
- **container**
    - 이미지를 실행할 경우 컨테이너라고 함 (ex. Process)
- pull
    - 도커 허브에서 이미지를 다운로드 받는 행위
- run
    - 이미지를 실행 시키는 행위

### 도커 허브: 이미지 다운로드

아파치 웹 서버인 httpd를 다운로드 해보자.

1. [https://hub.docker.com/](https://hub.docker.com/) 접속
2. Explore 메뉴 클릭
3. Containers 탭 클릭
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/602b823a-82b5-495f-aa8c-97212cc565cc/Untitled.png)
    
4. 아파치 웹 서버 httpd 클릭
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/74d6c9a9-48ad-44ed-8cd4-b88213eb6f50/Untitled.png)
    
    - 화면 오른쪽 명령어를 통해 설치 가능하다.
    - 참고: [Docker Referencce](https://docs.docker.com/reference/) 페이지를 통해 여러 메뉴얼을 확인 할 수 있다.
5. CLI에서 `docker pull httpd` 실행

### 도커 이미지 확인

- `docker images [OPTIONS] [REPOSITORY[:TAG]]`

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bb495898-da21-4dbf-8a53-6a984e26fb2f/Untitled.png)

## 이미지, 컨테이너 기본 조작 명령어

### 컨테이너 생성: run

- 새로운 컨테이너를 **생성**하는 명렁어
- 생성하는 동시에 실행 된다.

**GUI 사용**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3cfc4ea1-b86e-4fac-b43f-fda01ca3949c/Untitled.png)

1. images 탭 클릭
2. httpd 이미지의 run 버튼 클릭
3. 사용할 컨테이너 이름, 포트 번호, 등 옵션 정보 입력후 run 버튼으로 생성 및 실행
4. Containers / Apps 탭에서 해당 이미지를 클릭하여 컨테이너의 정보 확인 가능
    - LOGS: 컨테이너의 로그 표시
    - INSPECT: 환경, Port 표시
    - STATS: 통계 표시

**CLI 환경에서 사용**

- `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/840d770d-546e-4fee-9966-3e18b35c5b65/Untitled.png)

- 이름을 명시 하지 않을 경우 임의의 이름으로 컨테이너가 만들어 지게 된다.
- `docker --name ws2 httpd`
    - httpd 이미지를 ws2 이름이라는 컨테이너로 실행

### 컨터이너 정보: ps

- **`docker ps`**
    - 현재 실행 중인 컨테이너 정보를 보여주는 커맨드 명령어
    - 옵션
        - `-a` : 모든 컨테이너의 정보를 보여줌 (종료된 컨테이너 또한)

### 컨테이너 종료: stop

- `docker stop [OPTIONS] CONTAINER [CONTAINER...]`

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db91fae0-80bb-4ebd-9202-04319913fe33/Untitled.png)

- docker stop 명령어는 컨테이너를 종료하는 것이지 삭제 하는 것이 아니다.

### 컨테이너 실행: start

- `docker start [OPTIONS] CONTAINER [CONTAINER...]`
- stop 상태의 컨테이너를 실행 시키는 명렁어

### 컨테이너 로그: logs

- `docker logs [OPTIONS] CONTAINER`
- 실행 중인 docker의 로그를 확인하는 커맨드
- 옵션
    - `--follow`, `-f` : 로그를 실시간으로 보여준다. (로그 기록 추종)

### 컨테이너 삭제: rm

- `docker rm [OPTIONS] CONTAINER [CONTAINER...]`
- 컨테이너를 영구적으로 삭제하는 명령어
- 실행중인 컨테이너는 지울 수 없기 때문에 종료 후 삭제해야한다.

### 이미지 삭제: rm

- `docker rmi [OPTIONS] IMAGE [IMAGE...]`
- 도커 허브에서 다운로드 받은 이미지를 영구적으로 삭제하는 명령어

## 네트워크

### 도커 사용 X

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2bc578a0-b619-440e-ac4b-724ca72572b3/Untitled.png)

- 두대의 컴퓨터가 필요하다. (클라이언트, 서버)
- **클라이언트**
    - 웹 브라우저
- **서버**
    - 웹 서버
    - 파일 시스템: 데이터가 저장되는 공간
- 클라이언트가 서버의 포트를 통해 접속하여 파일 시스템의 HTML 파일을 보여준다.

### 도커 사용

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/079db46b-d068-4fc8-851a-c6f67e2f9956/Untitled.png)

- 웹서버가 컴퓨터가 아닌 컨테이너에 설치된다.
- 컨테이너가 설치된 운영체제를 도커 호스트(Host)라고 한다.
    - 하나의 호스트에는 여러개의 컨테이너가 만들어질 수 있다.
- 컨테이너와 호스트 모두 독립적인 실행환경으로 각자 독립적인 포트와 파일 시스템을 가지고 있다.
- 클라이언트에서 컨테이너로 접근하기 위해서는 호스트와 컨테이너의 포트를 연결 시켜줘야 한다.
- 접속 시작
    - 호스트의 포트 (ex.8080) 와 컨테이너의 포트 (ex. 80) 를 연결
        - `docker run -p 8080:80 httpd`
            - 호스트 포트 : 컨테이너 포트
        - port forwarding

### 실습 - GUI 사용

1. Images → Run
2. Local Host: Host의 Port 번호
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/01175548-aade-40e3-9aa5-3c93275cdd2b/Untitled.png)
    

### 실습 - CLI 사용

- `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`
- 옵션
    - `--publish`, `-p`: 컨테이너의 포트를 호스트로 공개(연결) 한다.
- 예제 코드
    
    ```bash
    # ws3 httpd 이미지 생성
    docker run --name ws3 -p 8081:80 httpd
    
    # 생성된 이미지 확인
    docker ps
    ```
    

### 접속 확인

- `localhost:[포트번호]` 에 접속하여 정상 동작 여부 확인 가능

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c8c399a-b142-4295-b7d4-f68a79cab1a9/Untitled.png)

## 명령어(CLI) 사용

- 실행 중인 컨테이너의 CLI을 사용할 수 있다.

### GUI 사용

- Containers / Apps 탭에서 ws2 컨테이너의 CLI 버튼을 통해 해당 컨테이너의 CLI창을 사용할 수 있다.

### CLI 사용

- `docker exec [OPTIONS] CONTAINER  COMMAND [ARG...]`
- 컨테이너의 쉘 사용
    - `docker exec -it CONTAINER /bin/sh`

## 📙 REFERENCE

- 생활코딩 Docker 입구 수업 ([https://opentutorials.org/course/4781](https://opentutorials.org/course/4781))
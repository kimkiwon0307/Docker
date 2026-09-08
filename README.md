<details>
<summary><h2>1장 도커란?</h2></summary>

---

### 1.1 가상 머신과 도커 컨테이너

* **가상 머신이란?**
  * 하나의 물리 서버 안에 여러 개의 가상 컴퓨터를 만드는 기술
  * **강한 독립성:** 다른 VM에 영향을 주지 않음
  * 서로 다른 OS 사용 가능
  * **단점:** 시스템 자원 낭비 발생

* **컨테이너의 등장**
  * 가상 머신보다 가볍고 빠름
  * 가장 큰 차이는 컨테이너마다 OS 전체를 설치하지 않는다는 점
  * 컨테이너는 Host OS의 커널을 공유함
  * **비유:** 가상 머신은 각각의 독립된 집을 짓는 것이고, 컨테이너는 하나의 큰 건물 안에 여러 개의 독립된 공간을 만드는 것과 비슷함

* **도커(Docker)**
  * 컨테이너를 쉽게 만들고 관리할 수 있도록 도와주는 도구

---

### 1.2 도커를 시작해야 하는 이유

* **1.2.1 애플리케이션의 개발과 배포가 편해진다.**
  * 애플리케이션과 실행 환경을 함께 묶을 수 있음
  * 개발 환경, 테스트 환경, 운영 환경을 최대한 동일하게 구축 가능

* **1.2.2 여러 애플리케이션의 독립성과 확장성이 높아진다.**
  * 각 애플리케이션을 독립적인 컨테이너로 실행 가능
  * 시스템 확장 및 관리가 용이해짐

* **1.2.3 Docker를 통해 컨테이너 기술을 쉽게 학습할 수 있다.**
  * 컨테이너 기술 자체는 Docker만의 기술이 아님
  * 복잡한 컨테이너 기술을 직관적인 명령어로 쉽게 사용할 수 있게 지원

---

### 1.3 도커 엔진 설치

* **1.3.1 Docker Engine의 종류 및 버전**
  * **Docker Engine이란?** Docker를 실제로 실행하는 핵심 프로그램 (`docker run`, `docker ps`, `docker images` 등)
  * **Docker Engine 구성 요소:**
    * **Docker CLI:** 사용자가 명령어를 입력하는 인터페이스 부분
    * **Docker Daemon:** Docker의 핵심 백그라운드 프로세스
    * **Docker API:** Docker CLI와 Docker Daemon 사이의 통신을 담당

</details>


< details > < summary > < h2 > 02장 도커 엔진 </ h2 > </ summary >

---

### 2.1 도커 이미지와 컨테이너
* 소스코드 -> Image -> Container 이 흐름을 이해하는 것이 Docker의 시작이다.
* **2.1.1 도커 이미지**
  * Docker 이미지는 컨테이너를 만들기 위한 실행 가능한 템플릿(설계도)이다.
  * **비유로 이해하기:** 이미지는 붕어빵 틀이다. (이미지는 원본 템플릿이고 컨테이너는 실제로 만들어진 실행 결과물이다.)
  * 도커 이미지 확인하기 : ` docker images ` 또는 ` docker image ls `
  * 항목 : REPOSITORY(이미지이름), TAG(이미지 버전), IMAGE ID(이미지 식별자), CREATED(생성 시간), SIZE(이미지 크기)
  * Image 이름과 Tag : ` nginx:latest ` = 이미지이름:버전
  * Docker Hub에서 이미지 다운로드 : Docker는 필요한 이미지를 다운로드해서 사용할 수 있다.
  * Docker Hub란? : Docker 이미지를 저장하고 공유하는 이미지 레지스트리이다.
* **2.1.2 Docker Container**
  * Docker Container는 Docker 이미지를 실제로 실행한 상태이다.
  * 컨테이너 실행 : ` docker run hello-world ` (내 pc에 이미지가 있으면 컨테이너 생성, 없으면 hub에서 다운로드 후 컨테이너 생성 후 실행)
  * Nginx 컨테이너 실행 : ` docker run nginx ` (이렇게 실행하면 현재 터미널이 컨테이너 로그에 연결될 수 있어서 보통 백그라운드로 실행한다: ` docker run -d nginx `)
  * 실행 중인 컨테이너 확인 : ` docker ps ` (IMAGE는 어떤 이미지를 사용했는지 보여주고, CONTAINER ID는 실행된 컨테이너를 구분하는 ID이다)
  * 모든 컨테이너 확인 : ` docker ps -a ` (실행이 종료된 컨테이너까지 모두 확인한다)
  * Image 하나로 여러 Container 실행 가능하다.
  * 컨테이너 이름 지정하기 : ` docker run -d --name web-server nginx `
  * 컨테이너 중지하기 : ` docker stop web-server ` 또는 ` docker stop Container ID `
  * 중지된 컨테이너 다시 실행하기 : ` docker start web-server ` (` docker run `은 새로운 컨테이너 생성이고 ` start `는 기존 컨테이너 다시 실행이다)
  * 컨테이너 삭제 : ` docker stop web-server ` 후 ` docker rm web-server `

---

### 2.2 도커 컨테이너 다루기
* **2.2.1 컨테이너 생성**
  * 컨테이너를 생성하고 실행하는 명령어 : ` docker run 이미지이름 `
  * ` docker run `은 컨테이너를 생성하고 실행한다는 뜻이다.
  * hello-world 실행 : ` docker run hello-world ` (Docker가 hello-world 이미지를 다운로드하고 컨테이너를 실행한다)
  * 백그라운드에서 컨테이너 실행하기 : ` docker run -d nginx ` (`-d`는 detached mode : 터미널과 분리해서 백그라운드 실행한다는 의미)
  * 컨테이너 이름 지정하기 : ` docker run -d --name my-nginx nginx `
* **2.2.2 컨테이너 목록 확인**
  * ` docker ps `는 현재 실행 중인 컨테이너만 보여준다.
  * CONTAINER ID (컨테이너 식별 번호), IMAGE(사용한 이미지), COMMAND(컨테이너 실행 명령), CREATED(생성 시간), STATUS(현재 상태), PORTS(포트 정보), NAMES(컨테이너 이름)을 확인할 수 있다.
  * 모든 컨테이너 확인 : ` docker ps -a ` 또는 ` docker container ls -a `
  * 컨테이너 상태 이해하기 : Created(생성됨), Up(실행 중), Exited(종료됨)
* **2.2.3 컨테이너 삭제**
  * 컨테이너는 보통 실행 중이면 바로 삭제할 수 없다. 먼저 중지하고 삭제한다. (` docker stop my-nginx ` -> ` docker rm my-nginx `)
  * 실행 중인 컨테이너를 강제로 삭제하기 : ` docker rm -f my-nginx `
  * Container ID로 작업하기 : ` docker stop Container ID ` -> ` docker rm Container ID `

</ details >







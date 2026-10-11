<details>
<summary><b>1장 도커란?</b></summary>

## Docker와 Container
VM은 각각 자기 Guest OS와 Kernel을 가진다.
Container는 Host의 Kernel을 공유한다.
Container가 VM보다 가볍고 빠르게 시작되고 자원 사용량이 적다 라는 장점이 있다.
VM은 OS 자체를 분리하는 데 강하다
Container는 애플리케이션 실행 환경을 격리하는 데 강하다.

## Docker는 왜 사용할까?
Docker를 사용하면 애플리케이션과 필요한 실행환경을 Image로 묶어서 실행할 수 있다.
환경 차이를 줄이고 애플리케이션을 일관되게 실행하기 위해 Docker를 사용한다.

## Docker Engine
Docker Engine은 Container를 실제로 관리하는 핵심 프로그램이다.
사용자 → docker 명령어 → Docker Engine → Image / Container / Network / Volume 관리 흐름으로 동작한다.

## 확인 문제
### 1. VM과 Container의 가장 큰 차이는 무엇인가요?
- **정답:** VM은 각각 Guest OS와 Kernel을 가지지만, Container는 Host의 Kernel을 공유합니다.

---

### 2. Container가 VM보다 일반적으로 가벼운 이유는 무엇인가요?
- **정답:** 별도의 Guest OS 없이 Host Kernel을 공유하기 때문에 시작이 빠르고 자원 사용량이 적습니다.

---

### 3. Docker를 사용하는 대표적인 이유를 하나 말해보세요.
- **정답:** 애플리케이션과 필요한 실행 환경을 Image로 묶어 개발환경과 운영환경의 차이를 줄이고 일관되게 실행하기 위해 사용합니다.

---

### 4. Docker Engine은 무슨 역할을 하나요?
- **정답:** Container, Image, Network, Volume 등을 생성하고 관리하는 Docker의 핵심 프로그램입니다.

---

### 5. Container마다 독립된 Guest OS가 있나요?
- **정답:** 아니요. Container는 Host의 Kernel을 공유하며 일반적으로 별도의 Guest OS를 가지지 않습니다.

</details>

<details>
<summary><b> 2장 도커 엔진</b></summary>


## Image와 Container
1. Image는 Container를 만들기 위한 읽기 전용 템플릿이다. 
2. Container는 Image를 기반으로 생성된 실제 실행 환경이다.
3. Image는 Read only, Container는 Image + Writable Layer 이다.

## create / start / run
1. docker create는 Container를 생성만 한다. docker create --name web01 nginx
2. docker start는 container를 실행 한다. docker start web01
3. docker run은 생성과 실행을 한다. docker run -d --name web01 nginx

## Container 상태 확인
1. 실행 중인 Container : docker ps
2. 실행 중 + 정지된 Container : docker ps -a

## Port Mapping
1. Container 안에서 nginx가 80포트로 실행되고 있어도, Host 밖에서 바로 접근할 수 있는 건 아니다.
2. docker run -d --name web01 -p 8080:80 nginx
3. Host 8080 → Container 80 → nginx
4. -p HostPort:ContainerPort

## docker exec
1. 실행 중인 Container 안에서 명령을 실행하고 싶을 때 사용한다.
2. docker exec web01 nginx -v : web01 Container에서 nginx -v를 실행하는 것이다.
3. docker exec -it web01 /bin/bash : Shell로 들어가기
4. docker exec : 실행 중인 Container 선택, 그 안에서 명령 실행
5. -it : i는 입력을 받을 수 있게, t는 터미널 형태로 사용
6. docker exec -it 컨테이너이름 /bin/bash

## Volume
1. Container 내부에 데이터를 저장하면 기본적으로 그 데이터는 Container의 Writable Layer에 저장된다.
2. Container 자체를 삭제하면 Writable Layer도 같이 사라질 수 있다. DB 데이터나 업로드 파일처럼 계속 보존해야 하는 데이터는 Volme을 사용한다.
3. Volume은 Container의 생명주기와 데이터를 분리해서, Container를 삭제해도 데이터를 유지할 수 있게 해주는 저장공간이다.
4. Contaienr → (mount) → volume(데이터유지)
5. docker run -d --name mysql01 -v mysql-data:/var/lib/mysql mysql
6. mysql-data : Docker Volume 이름, /var/lib/mysql : Container 내부 경로
7. -v Volume이름:Container경로
8. 데이터가 Container와 Volume 양쪽에 복사되는 게 아니라 Volume을 바라보는 Mount Point가 된다.
9. 보존해야 하는 애플리케이션 데이터가 실제 저장되는 경로에 Volume을 Mount한다.

## Volume 종류
1. Named Volume : 이름을 직접 지정 : -v mysql-data:/var/lib/mysql
2. Anonymous Volume : 이름을 직접 지정하지 않는다. : -v /var/lib/mysql : Docker가 임의의 이름을 만들어 관리
3. Bind Mount : Host의 실제 디렉터리를 Container에 연결 : -v /home/ubuntu/data:/data

## Volume 기본 명령
1. docker volume ls : Volume 목록
2. docker volume create mysql-data : Volume 생성
3. docker volume inspect mysql-data : volme 상세 확인
4. docker volume rm mysql-data : 삭제

## Container 삭제와 Volume
1. Container를 삭제하더라고 Volume은 유지
2. 새 Container를 만들어 같은 Volume을 다시 연결하면 기존 데이터를 다시 사용할 수 있다.

## Docker Network
1. Container끼리 통신하려면 네트워크가 필요하다.
2. 같은 네트워크에서 Container들 끼리 통신할 때 IP를 직접 몰라도 된다. : docker exec web curl app
3. Container 이름으로 접근할 수 있다.
4. Docker가 같은 네트워크 안에서 이름을 IP로 바꿔주는 DNS 기능을 제공하기 때문이다.
5. Container IP는 재생성되면 바뀔 수 있다. 그래서 이름을 사용한다.

## Docker Network 기본 명령
1. docker network ls : Network 목록
2. docker network inspect app-net : 상세 확인
3. docker network create app-net : 생성
4. docker network rm app-net : 삭제

## 기본 bridge와 User-defined bridge
1. Docker 설치 후 기본적으로 bridge Network가 있다.
2. 같은 User-defined Network에 있는 Container끼리는 이름으로 통신할 수 있다.

## 로그 확인
1. docker logs 컨테이너이름
2. docker logs -f web01 : 실시간 보기

## Resource 확인
1. Container도 CPU와 Memory를 사용한다.
2. docker stats : 현재 사용량 확인

## CPU / Memory 제한
1. Container가 Host 자원을 너무 많이 사용하지 못하도록 제한할 수도 있다.
2. docker run -d --name limit-test --cpus="0.5" --memory="128m" nginx

## inspect 
1. Container의 상세 설정을 보고 싶을 때
2. docker inspect web01
3. Network 설정, IP, Volume, 환경변수, Resource 제한, Container 상태 등을 볼 수 있다.

## Docker Image 관리
1. docker images 또는 docker image ls
2. docker pull nginx:latest : Registry에서 image 가져오기
3. docker rmi nginx:latest : Image 삭제

## docker run과 pull 관계
1. docker run nginx 를 실행하면 Docker 가 먼저 Image가 있는지 확인한다.
2. 로컬 Image 없음 → Registry에서 pull → Container 생성 → 실행

## Image Layer
1. Image는 보통 여러 Layer로 구성된다.
2. docker history nginx : 어떤 Layer들로 구성됐는지 확인할 때 사용

## docker commit
1. nginx Image → Container 생성 → Container 내부 파일 수정
2. 변경된 Container 상태를 새로운 Image로 만들 수 있다.
3. docker commit web01 my-nginx:v1
4. 실무에서는 재현성과 자동화 때문에 Dockerfile을 사용한다.

## Image를 파일로 저장
1. Image를 tar 파일로 내보낼 수 있다.
2. docker save -o my-nginx.tar my-nginx:v1
3. docker load -i my-nginx.tar

## Registry와 Image
1. Docker Hub 같은 Registry를 사용하면 Image를 원격 저장소에 올리고 내려받을 수 있다.
2. 내 pc → (push) → Registry → (pull) → 다른 서버
3. docker push 이미지이름
4. docker pull 이미지이름

## Image Tag
1. nginx:1.26
2. nginx : Image 이름
3. 1.26 : Tag
4. Tag는 Image의 버전이나 변형을 구분할 때 사용

## Dockerfile
1. Docker Image를 어떻게 만들지 코드로 적어놓은 파일
2. Dockrfile은 무슨 Base Image를 썼는지? 어떤 파일을 복사했는지, 어떤 명령을 실행했는지, 어떤 프로그램을 실행할지 를 코드로 남길 수 있다.

## 기본 구조
1. FROM nginx:latest
   COPY index.html /usr/share/nginx/html/
2. nginx Image를 기반으로 index.html을 Image 안으로 복사해서 새 Image 생성
3. Image Build : docker build -t my-nginx:v1 .
4. docker build : Image Build, -t my-nginx:v1 : Image 이름과 Tag, . : 현재 디렉터리를 Build Context로 사용

## FROM
1. FROM nginx:latest
2. 어떤 Image를 기반으로 새 Image를 만들 것인가?

## COPY
1. COPY index.html /usr/share/nginx/html/
2. 현재 Build Context의 index.html을 Image 내부 /usr/share/nginx/html로 복사한다.

## RUN
1. RUN apt-get update
2. RUN은 Image를 Build하는 도중에 실행되는 명령이다.
3. RUN = Build 시점

## CMD
1. CMD는 Container가 실행될 때 사용할 기본 명령이나 인자이다.
2. CMD ["nginx", "-g", "daemon off;"]

## ENTRYPOINT
1. ENTRYPOINT는 Container에서 실행할 주 프로그램을 고정하는 용도
2. ENTRYPOINT ["ping"]
   CMD ["localhost"]
3. 기본으로 Container를 실행하면 ping localhost가 된다.
4. ENTRYPOINT : 주 실행 프로그램
5. CMD : 기본 명령/인자

## WORKDIR
1. WORKDIR /app
2. 이후 Dockerfile 명령들이 작업할 기본 디렉터리를 지정

## ENV
1. 환경변수를 설정
2. ENV APP_ENV=production

## EXPOSE
1. EXPOSE 80
2. 이 Container가 보통 80번 Port를 사용한다는 정보를 기록

## ADD
1. ADD도 파일을 Image에 넣을 수 있다.
2. 일반적인 파일 복사는 COPY를 우선 사용
3. ADD는 tar 자동 압축 해제 같은 추가 기능이 있기 때문에 특별한 이유가 있을 때 사용한다.

## 확인문제

1. Docker Image란 무엇인가요?
- **정답:** Container를 생성하기 위한 읽기 전용 템플릿입니다.

2. Docker Container란 무엇인가요?
- **정답:** Image를 기반으로 생성되어 실제 애플리케이션 프로세스가 실행되는 격리된 실행 환경입니다.

3. Container 안에서 파일을 수정하면 원본 Image도 수정되나요?
- **정답:** 아니요. Image는 읽기 전용이고 변경사항은 Container의 Writable Layer에 기록됩니다.

4. docker create, docker start, docker run의 차이는 무엇인가요?
- **정답:** create는 Container 생성, start는 생성된 Container 실행, run은 쉽게 생성과 실행을 함께 수행한다고 이해하면 됩니다.

5. docker ps와 docker ps -a의 차이는 무엇인가요?
- **정답:** docker ps는 실행 중인 Container를 보여주고, docker ps -a는 정지된 것을 포함한 모든 Container를 보여줍니다.

6. -p 8080:80의 의미는 무엇인가요?
- **정답:** Host의 8080번 Port로 들어온 요청을 Container의 80번 Port로 전달한다는 의미입니다.

7. 실행 중인 Container 내부에서 명령을 실행하려면 무엇을 사용하나요?
- **정답:** docker exec를 사용합니다. Shell 접속은 보통 docker exec -it 컨테이너명 /bin/bash 형태로 사용합니다.

8. Docker Volume을 사용하는 이유는 무엇인가요?
- **정답:** Container와 데이터의 생명주기를 분리하여 Container를 삭제해도 필요한 데이터를 유지하기 위해 사용합니다.

9. -v mysql-data:/var/lib/mysql에서 각각 무엇을 의미하나요?
- **정답:** mysql-data는 Docker Volume 이름이고, /var/lib/mysql은 Container 내부의 Mount 경로입니다.

10. Named Volume과 Bind Mount의 차이는 무엇인가요?
- **정답:** Named Volume은 Docker가 저장 위치를 관리하고, Bind Mount는 사용자가 Host의 특정 디렉터리를 직접 지정합니다.

11. 같은 User-defined Docker Network에 있는 Container끼리는 어떻게 통신할 수 있나요?
- **정답:** Docker DNS를 이용해 Container 이름으로 서로 통신할 수 있습니다.

12. 같은 Docker Network 내부의 Container끼리 통신할 때 반드시 -p가 필요한가요?
- **정답:** 아니요. -p는 주로 Host에서 Container로 접근할 때 사용하며, 같은 Network의 Container끼리는 내부 Network로 직접 통신할 수 있습니다.

13. Container 로그를 확인하는 명령은 무엇인가요?
- **정답:** docker logs 컨테이너명입니다. 실시간으로 보려면 docker logs -f 컨테이너명을 사용합니다.

14. Container의 CPU와 Memory 사용량을 확인하는 명령은 무엇인가요?
- **정답:** docker stats입니다.

15. Container의 상세 설정을 확인하는 명령은 무엇인가요?
- **정답:** docker inspect 컨테이너명입니다.

16. Container의 CPU와 Memory를 제한하려면 어떤 옵션을 사용할 수 있나요?
- **정답:** CPU는 --cpus, Memory는 --memory 옵션을 사용할 수 있습니다.

17. Registry에서 Image를 가져오는 명령은 무엇인가요?
- **정답:** docker pull입니다.

18. Image를 삭제하는 명령은 무엇인가요?
- **정답:** docker rmi입니다.

19. docker save/load와 docker push/pull의 차이는 무엇인가요?
- **정답:** save/load는 Docker Image와 tar 파일 사이에서 사용하고, push/pull은 Local Image와 Registry 사이에서 사용합니다.

20. Docker Image의 Tag는 무엇인가요?
- **정답:** Image의 버전이나 변형을 구분하기 위한 값입니다. 예를 들어 nginx:1.26에서 1.26이 Tag입니다.

21. docker commit은 무엇인가요?
- **정답:** 변경된 Container의 현재 상태를 새로운 Image로 만드는 명령입니다.

22. 실무에서 docker commit보다 Dockerfile이 중요한 이유는 무엇인가요?
- **정답:** Image를 만드는 과정이 코드로 기록되므로 재현성, 변경 이력 관리, 자동화에 유리하기 때문입니다.

23. Dockerfile의 FROM은 무엇인가요?
- **정답:** 새로운 Image를 만들 때 사용할 Base Image를 지정합니다.

24. COPY는 무엇인가요?
- **정답:** Build Context의 파일이나 디렉터리를 Image 내부로 복사합니다.

25. RUN과 CMD의 차이는 무엇인가요?
- **정답:** RUN은 Image를 Build할 때 실행되고, CMD는 Container가 시작될 때 사용할 기본 명령이나 인자를 지정합니다.

26. ENTRYPOINT와 CMD의 차이를 간단히 설명하면?
- **정답:** ENTRYPOINT는 주 실행 프로그램을 지정하고, CMD는 기본 명령이나 기본 인자를 지정하는 데 주로 사용합니다.

27. WORKDIR은 무엇인가요?
- **정답:** 이후 Dockerfile 명령들이 실행될 기본 작업 디렉터리를 지정합니다.

28. ENV는 무엇인가요?
- **정답:** Image나 Container에서 사용할 환경변수를 설정합니다.

29. EXPOSE 80을 쓰면 Host의 80번 Port가 자동으로 열리나요?
- **정답:** 아니요. EXPOSE는 Container가 사용하는 Port 정보를 나타내는 역할이고, 실제 Port Mapping은 -p로 설정합니다.

30. docker build -t myimage:v1 . 에서 마지막 . 은 무엇인가요?
- **정답:** 현재 디렉터리를 Docker Build Context로 사용한다는 뜻입니다.

</details>

<details>
<summary><b> 3장 Docker Swarm </b></summary>

## Docker Swarm 이란?
1. Docker만 단독으로 사용하면 보통 한 서버 안의 Container를 관리한다.
2. 서버가 여러 대가 되면 직접 Container를 하나씩 관리하기 어려워진다.
3. 이 여러 서버를 묶어서 하나의 Cluster처럼 관리하는 것이 Docker Swarm이다.
4. Docker Swarm은 여러 Docker 서버의 Container를 배치,확장,복구하는 Orchestration 기능이다.

## Node / Manager / Worker
1. Swarm에 참여하는 서버 한 대를 Node라고 한다.
2. Node는 크게 Manager와 Worker로 나뉜다.

## Manager Node
1. Manager는 Cluster를 관리한다.
2. Manager : Service 생성, 상태 관리, Worker에 Task 배치, Desired State 유지
3. Manager는 지시하고 관리하는 서버이다.

## Worker Node
1. Worker는 Manager가 배정한 작업을 실제로 실행한다.
2. Manager → Task 전달 → Worker → Container 실행
3. Worker는 실제 Container를 실행하는 서버이다.

## Service / Task / Replica
1. Service는 어떤 Container를 몇 개 실행할지 정의한 것이다.
2. nginx Service
   Image = nginx
   Replicas = 3
3. Swarm이 Container 3개를 Cluster에 배치한다.

## Replica
1. Replica는 원하는 실행 개수이다.
2. replicas = 3 이면 Container가 3개가 실행되도록 유지한다.

## Task
1. Task는 Service가 Node에 실제로 배정한 하나의 작업 단위이다.

## Desired State
1. Desired State는 사용자가 원하는 상태이다.
2. Desired State와 Current State를 비교해서 맞춰나간다.
3. 이 개념은 Kubernetes에서도 등장한다.

## Service 생성과 Scale
1. docker service create --name web --replicas 3 nginx
2. docker service ls : Service 확인
3. docker service ps web : Task 확인
4. docker service scale web=5 : Replica 증감

## 장애 자동 복구
1. Swarm의 장점 중 하나는 Container 하나가 죽어도 원하는 개수를 자동으로 복구할 수 있다.

## Rolling Update
1. 기존 Container를 점진적으로 새 버전 Container로 교체

## Rollback
1. 이전 정상 버전으로 되돌리기

## Node Drain
1. 어떤 Woker 서버를 점검해야 할 때 Node를 Drain 상태로 둘 수 있다.
2. Drain = 해당 Node를 유지보수하기 위해 Service Task 배치 대상에서 제외

## 확인문제
## Docker Swarm 확인문제

1. Docker Swarm이란 무엇인가요?
- **정답:** 여러 Docker 서버를 하나의 Cluster처럼 묶어 Container를 배치, 확장, 복구하는 Orchestration 기능입니다.

2. Node란 무엇인가요?
- **정답:** Docker Swarm Cluster에 참여하는 서버 한 대를 의미합니다.

3. Manager Node의 역할은 무엇인가요?
- **정답:** Cluster 상태를 관리하고 Service와 Task를 제어하며 Worker Node에 작업을 배치합니다.

4. Worker Node의 역할은 무엇인가요?
- **정답:** Manager가 배정한 Task를 받아 실제 Container를 실행합니다.

5. Service란 무엇인가요?
- **정답:** 어떤 Image를 사용해 Container를 몇 개 실행할지 정의한 Swarm의 관리 단위입니다.

6. Replica란 무엇인가요?
- **정답:** Service에서 원하는 Container의 실행 개수를 의미합니다.

7. Task란 무엇인가요?
- **정답:** Service에 의해 Node에 실제로 배정되는 하나의 실행 작업 단위입니다.

8. Desired State란 무엇인가요?
- **정답:** 사용자가 원하는 최종 상태를 의미하며, Swarm은 현재 상태를 Desired State에 맞추도록 계속 조정합니다.

9. replicas=3인데 Container 하나가 죽으면 어떻게 되나요?
- **정답:** Swarm이 현재 개수가 부족한 것을 감지하고 새 Task와 Container를 생성하여 다시 3개를 유지합니다.

10. docker service create --name web --replicas 3 nginx는 무엇을 의미하나요?
- **정답:** web이라는 이름의 Service를 만들고 nginx Image 기반 Container를 3개 유지하도록 설정한다는 의미입니다.

11. Service 목록을 확인하는 명령은 무엇인가요?
- **정답:** docker service ls입니다.

12. 특정 Service의 Task 상태를 확인하는 명령은 무엇인가요?
- **정답:** docker service ps 서비스이름입니다.

13. Service의 Replica 개수를 변경하는 것을 무엇이라고 하나요?
- **정답:** Scale이라고 합니다.

14. Service를 3개에서 5개로 늘리려면 어떤 식으로 명령할 수 있나요?
- **정답:** docker service scale web=5 같은 형식으로 사용할 수 있습니다.

15. Rolling Update란 무엇인가요?
- **정답:** 기존 Container를 한꺼번에 모두 교체하지 않고 새 버전으로 점진적으로 교체하는 방식입니다.

16. Rolling Update를 사용하는 이유는 무엇인가요?
- **정답:** 업데이트 중 서비스 중단 가능성을 줄이기 위해 사용합니다.

17. Rollback이란 무엇인가요?
- **정답:** 새 버전에 문제가 발생했을 때 이전 정상 버전으로 되돌리는 것입니다.

18. Node Drain이란 무엇인가요?
- **정답:** 특정 Node를 유지보수하기 위해 Service Task 배치 대상에서 제외하는 상태입니다.

19. Node Drain은 언제 사용하나요?
- **정답:** 서버 점검, 재부팅, 패치, 장애 대응 등으로 특정 Node에서 작업을 빼야 할 때 사용합니다.

20. Docker Swarm에서 자동 복구가 가능한 이유는 무엇인가요?
- **정답:** Swarm이 Desired State와 Current State를 계속 비교하고 차이가 있으면 필요한 Task를 다시 생성하기 때문입니다.

21. Docker 단독 실행과 Docker Swarm의 차이는 무엇인가요?
- **정답:** Docker 단독 실행은 주로 한 서버의 Container를 직접 관리하고, Swarm은 여러 서버를 Cluster로 묶어 Container를 자동 배치하고 관리합니다.

22. Manager와 Worker의 차이를 한 줄로 설명하면?
- **정답:** Manager는 Cluster를 관리하고 작업을 배치하며, Worker는 실제 Container를 실행합니다.

23. Service, Task, Container의 관계는 무엇인가요?
- **정답:** Service가 원하는 실행 상태를 정의하고, 그에 따라 Task가 Node에 배정되며, Task를 통해 실제 Container가 실행됩니다.

24. Swarm의 Desired State 개념은 Kubernetes에서 어떤 개념과 연결되나요?
- **정답:** ReplicaSet이나 Controller가 원하는 Pod 개수를 유지하는 Desired State 개념과 연결됩니다.

25. Docker Swarm의 Rolling Update는 Kubernetes의 어떤 기능과 연결되나요?
- **정답:** Kubernetes Deployment의 Rolling Update와 연결됩니다.

</details>

<details>
<summary><b> 4장 Docker Compose </b></summary>

## Docker Compose
1. 여러 Container로 구성된 애플리케이션을 하나의 YAML 파일로 정의하고 함께 실행,관리하는 도구이다.
2. Compose를 사용하면 compose.yaml 하나에 묶을 수 있다.

## 주요 명령
1. docker compose up -d : Container 생성, 실행
2. docker compose ps : Compose로 실행한 Container 상태 확인
3. docker compose stop : 중지
4. docker compose start : 시작
5. docker compose down : Container와 Compose Network 등을 정리
6. docker compose down -v : Volume까지 삭제

## Compose Network
1. 같은 Service 이름으로 통신할 수 있다.
2. Compose 안에서도 Container IP 보다 Service 이름으로 통신하는게 핵심이다.

## Compose Volume 
1. image: mysql
   volumes:
      - db-data:/var/lib/mysql

volumes:
   db-data:
2. 이렇게 하면 MySQL Container가 삭제돼도 db-data Volume에 DB 데이터를 유지할 수 있다.

## Compose Build
1. 이미 만들어진 Image만 사용하는 게 아니라 Dockerfile로 직접 Build할 수도 있다.

## 확인문제

1. Docker Compose란 무엇인가요?
- **정답:** 여러 Container로 구성된 애플리케이션을 하나의 YAML 파일로 정의하고 함께 실행·관리하는 도구입니다.

2. Docker Compose를 사용하는 이유는 무엇인가요?
- **정답:** 여러 Container를 각각 docker run으로 실행하지 않고 하나의 설정 파일로 일괄 관리하기 위해 사용합니다.

3. Compose 파일에서 services는 무엇인가요?
- **정답:** 실행할 Container들을 정의하는 영역입니다.

4. docker compose up -d는 무엇을 하나요?
- **정답:** Compose 파일에 정의된 Container들을 생성하고 백그라운드로 실행합니다.

5. docker compose ps는 무엇을 하나요?
- **정답:** Compose로 실행한 Container들의 상태를 확인합니다.

6. docker compose stop과 start의 역할은 무엇인가요?
- **정답:** stop은 Container를 중지하고, start는 중지된 Container를 다시 실행합니다.

7. docker compose down은 무엇을 하나요?
- **정답:** Compose로 생성한 Container와 Network 등을 정리합니다.

8. docker compose down -v는 무엇이 다른가요?
- **정답:** Container와 Network뿐 아니라 Volume까지 삭제합니다.

9. DB를 사용하는 환경에서 docker compose down -v를 조심해야 하는 이유는 무엇인가요?
- **정답:** DB 데이터가 저장된 Volume까지 삭제되어 데이터가 사라질 수 있기 때문입니다.

10. Docker Compose에서 Container끼리는 어떻게 통신할 수 있나요?
- **정답:** Compose가 만든 Network 안에서 Service 이름을 이용해 서로 통신할 수 있습니다.

11. Compose에서 Container IP보다 Service 이름을 사용하는 이유는 무엇인가요?
- **정답:** Container IP는 변경될 수 있지만 Service 이름은 설정에서 안정적으로 사용할 수 있기 때문입니다.

12. Compose의 Network 기능은 Docker의 어떤 개념과 연결되나요?
- **정답:** User-defined Docker Network와 Docker DNS 개념과 연결됩니다.

13. Compose에서 Volume을 사용하는 이유는 무엇인가요?
- **정답:** Container가 삭제되어도 DB 데이터나 필요한 파일을 유지하기 위해 사용합니다.

14. MySQL에서 /var/lib/mysql에 Volume을 연결하는 이유는 무엇인가요?
- **정답:** MySQL의 실제 데이터가 저장되는 경로이기 때문입니다.

15. Compose에서 build 옵션은 무엇을 의미하나요?
- **정답:** 지정한 경로의 Dockerfile을 이용해 Image를 Build하고 그 Image로 Container를 실행한다는 뜻입니다.

16. image와 build의 차이는 무엇인가요?
- **정답:** image는 기존 Image를 사용하고, build는 Dockerfile을 이용해 새로운 Image를 Build해서 사용합니다.

17. nginx, Spring Boot, MySQL을 Compose로 구성한다면 각각 무엇이 되나요?
- **정답:** 각각 하나의 Service로 정의할 수 있습니다.

18. Dockerfile과 Docker Compose의 차이는 무엇인가요?
- **정답:** Dockerfile은 하나의 Image를 만드는 방법을 정의하고, Docker Compose는 여러 Container의 실행 구성을 정의합니다.

19. Docker Compose와 Docker Swarm의 차이는 무엇인가요?
- **정답:** Compose는 주로 한 Host에서 여러 Container 애플리케이션을 구성하고, Swarm은 여러 Docker 서버를 Cluster로 묶어 Container를 오케스트레이션합니다.

20. 프로젝트의 Docker 단계에서 Compose를 어떻게 활용할 수 있나요?
- **정답:** nginx, Spring Boot, MySQL 같은 여러 Container를 Network와 Volume까지 포함해 하나의 compose.yaml로 구성하고 실행할 수 있습니다.










</details>






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

[4장 Docker Compose - 확인문제 + 정답]

**1. Docker Compose란 무엇인가요?**
- **정답:** 여러 Container로 구성된 애플리케이션을 하나의 YAML 파일로 정의하고 함께 실행·관리하는 도구입니다.

**2. Docker Compose를 사용하는 이유는 무엇인가요?**
- **정답:** 여러 Container를 각각 docker run으로 실행하지 않고 하나의 설정 파일로 일괄 관리하기 위해 사용합니다.

**3. Compose 파일에서 services는 무엇인가요?**
- **정답:** 실행할 Container들을 정의하는 영역입니다.

**4. docker compose up -d는 무엇을 하나요?**
- **정답:** Compose 파일에 정의된 Container들을 생성하고 백그라운드로 실행합니다.

**5. docker compose ps는 무엇을 하나요?**
- **정답:** Compose로 실행한 Container들의 상태를 확인합니다.

**6. docker compose stop과 start의 역할은 무엇인가요?**
- **정답:** stop은 Container를 중지하고, start는 중지된 Container를 다시 실행합니다.

**7. docker compose down은 무엇을 하나요?**
- **정답:** Compose로 생성한 Container와 Network 등을 정리합니다.

**8. docker compose down -v는 무엇이 다른가요?**
- **정답:** Container와 Network뿐 아니라 Volume까지 삭제합니다.

**9. DB를 사용하는 환경에서 docker compose down -v를 조심해야 하는 이유는 무엇인가요?**
- **정답:** DB 데이터가 저장된 Volume까지 삭제되어 데이터가 사라질 수 있기 때문입니다.

**10. Docker Compose에서 Container끼리는 어떻게 통신할 수 있나요?**
- **정답:** Compose가 만든 Network 안에서 Service 이름을 이용해 서로 통신할 수 있습니다.

**11. Compose에서 Container IP보다 Service 이름을 사용하는 이유는 무엇인가요?**
- **정답:** Container IP는 변경될 수 있지만 Service 이름은 설정에서 안정적으로 사용할 수 있기 때문입니다.

**12. Compose의 Network 기능은 Docker의 어떤 개념과 연결되나요?**
- **정답:** User-defined Docker Network와 Docker DNS 개념과 연결됩니다.

**13. Compose에서 Volume을 사용하는 이유는 무엇인가요?**
- **정답:** Container가 삭제되어도 DB 데이터나 필요한 파일을 유지하기 위해 사용합니다.

**14. MySQL에서 /var/lib/mysql에 Volume을 연결하는 이유는 무엇인가요?**
- **정답:** MySQL의 실제 데이터가 저장되는 경로이기 때문입니다.

**15. Compose에서 build 옵션은 무엇을 의미하나요?**
- **정답:** 지정한 경로의 Dockerfile을 이용해 Image를 Build하고 그 Image로 Container를 실행한다는 뜻입니다.

**16. image와 build의 차이는 무엇인가요?**
- **정답:** image는 기존 Image를 사용하고, build는 Dockerfile을 이용해 새로운 Image를 Build해서 사용합니다.

**17. nginx, Spring Boot, MySQL을 Compose로 구성한다면 각각 무엇이 되나요?**
- **정답:** 각각 하나의 Service로 정의할 수 있습니다.

**18. Dockerfile과 Docker Compose의 차이는 무엇인가요?**
- **정답:** Dockerfile은 하나의 Image를 만드는 방법을 정의하고, Docker Compose는 여러 Container의 실행 구성을 정의합니다.

**19. Docker Compose와 Docker Swarm의 차이는 무엇인가요?**
- **정답:** Compose는 주로 한 Host에서 여러 Container 애플리케이션을 구성하고, Swarm은 여러 Docker 서버를 Cluster로 묶어 Container를 오케스트레이션합니다.

**20. 프로젝트의 Docker 단계에서 Compose를 어떻게 활용할 수 있나요?**
- **정답:** nginx, Spring Boot, MySQL 같은 여러 Container를 Network와 Volume까지 포함해 하나의 compose.yaml로 구성하고 실행할 수 있습니다.

</details>

<details>
<summary><b> 5장 Kubernetes 설치 </b></summary>

## Kubernetes 설치 환경의 종류
1. 로컬 학습용 : Minikube, Docker Desktop
2. 직접 구축 : kubeadm
3. Cloud 자동화 :  kops
4. Managed Kubernetes : GKE, EKS, AKS
5. 같은 Kubernetes를 어떤 방식으로 설치,운영 하느냐가 다르다.
6. Node는 Kubernetes Cluster에 참여한는 서버 한 대이다.
7. Control Plane도 Node이고 Worker도 Node이다.

## Kubernetes 버전 선택
1. 1.37.1
2. 1 은 Major, 37은 Minor, 1은 Patch

## 주요 도구
1. kubeadm : Cluster 생성 / Worker join : Cluster 구축
2. kubelet : 각 Node에서 Pod를 관리하는 Agent : Node 관리
3. Kubelctl : 사용자가 Kubernetes에 명령하는 CLI : 사용자 명령

## 개발 용도의 Kubernetes 설치
1. Docker Desktop, Minikube
2. Minikube는 로컬에서 Kubernetes Cluster를 쉽게 만들어 학습,개발할 수 있게 해주는 도구

## 여러 서버로 구성된 Kubernetes Cluster
1. 로컬 단일 환경이 아니라 여러 서버를 하나의 Cluster로 묶는 방식
2. control-plane, worker1, worker2

## kubeadm
1.  VM 3대 준비
2.  control-plane, worker1, worker2

## containerd
1. Kubernetes가 Container를 실제로 실행하려면 Container Runtime이 필요하다.
2. containerd를 사용했다.
3. Kubernetes → kubelet → containerd → Container

## Control Plane 핵심 구성요소

## kube-apiserver 
1. Kubernetes의 중심 요청 창구
2. kubectl → API Server → Kubernetes
3. 대부분의 Kubernetes 요청이 API Server를 통한다.

## etcd 
1. Cluster 상태를 저장하는 데이터베이스
2. 어떤 Node가 있는지, 어떤 Pod가 있는지, 원하는 상태가 무엇인지 같은 Cluster 정보를 저장

## kube-scheduler 
1. 새 Pod를 어느 Node에서 실행할지 결정

## kube-controller-manager
1. 원하는 상태와 현재 상태를 계속 맞춤

## Worker Node의 Kubelet
1. 각 Node에서는 Kubelet이 동작한다.
2. Kubelet은 자신의 Node에서 Pod가 원하는 상태로 실행되도록 관리한다.
3. API Server → kubelet → containerd → Pod / Container

## Flannel과 CNI
1. 처음 Cluster를 만들었을 때 Node가 NotReady 상태였다.
2. 그 이유 중 하나가 pod Network가 아직 구성되지 않았기 때문이다.
3. 그래서 Flannel을 설치했다.
4. Kubernetes Pod Network를 구성하는 CNI Plugin

## Node Network와 Pod Network
1. Node → 192.168.111.x
2. Pod → 10.244.x.x
3. 별도의 Network를 사용

## CoreDNS
1. Kubernetes 내부 DNS 역할을 한다.
2. Kubernetes 안에서 이름으로 Service나 Pod 관련 대상을 찾을 수 있도록 도와준다.

## kube-proxy
1. kube-proxy는 각 Node에서 Service Traffic이 Pod 쪽으로 전달될 수 있도록 네트워크 규칙을 관리한다.
2. Service 통신을 도와주는 네트워크 구성요소

## DaemonSet 
1. 각 Node마다 Pod 하나씩 실행하도록 하는 방식을 DaemonSet이라고한다.

## kops
1. AWS 같은 Cloud에서 인프라 생성과 Kubernetes Cluster 구축을 더 넓게 자동화하는 도구
2. kubeadm : 서버를 내가 먼저 준비, Kubernetes 설치
3. kops : Cloud 인프라 생성까지 더 많이 자동화

## GKE
1. Google Kubernetes Engine
2. 구글이 제공하는 Managed Kubernetes

## 확인 문제

**1. Kubernetes 설치 방식이 여러 개라는 것은 Kubernetes 종류가 여러 개라는 뜻인가요?**
- **정답:** 아닙니다. 같은 Kubernetes를 어떤 방식으로 설치하고 운영하느냐가 다른 것입니다.

**2. Node란 무엇인가요?**
- **정답:** Kubernetes Cluster에 참여하는 서버 한 대를 의미합니다.

**3. Minikube란 무엇인가요?**
- **정답:** 로컬 환경에서 Kubernetes Cluster를 쉽게 만들어 학습이나 개발에 사용할 수 있게 해주는 도구입니다.

**4. kubeadm이란 무엇인가요?**
- **정답:** Linux 서버에 Kubernetes Cluster를 직접 생성하고 Worker Node를 Join시키는 데 사용하는 공식 도구입니다.

**5. kubelet의 역할은 무엇인가요?**
- **정답:** 각 Node에서 Pod가 원하는 상태로 실행되도록 관리하는 Agent입니다.

**6. kubectl의 역할은 무엇인가요?**
- **정답:** 사용자가 Kubernetes API Server에 명령을 전달하는 CLI 도구입니다.

**7. kubeadm, kubelet, kubectl을 한 줄씩 구분하면?**
- **정답:**
  - kubeadm = Cluster 구축
  - kubelet = Node의 Pod 관리
  - kubectl = 사용자 명령

**8. Kubernetes 버전에서 1.37.1은 어떻게 구분하나요?**
- **정답:** 1은 Major, 37은 Minor, 1은 Patch입니다.

**9. Version Skew란 무엇인가요?**
- **정답:** Kubernetes 구성요소 간에 허용되는 버전 차이를 의미합니다.

**10. 우리가 kubeadm 실습에서 만든 Cluster 구성은 어떻게 되었나요?**
- **정답:** Control Plane 1대와 Worker Node 2대로 총 3 Node Cluster를 구성했습니다.

**11. Control Plane의 역할은 무엇인가요?**
- **정답:** Kubernetes Cluster의 전체 상태와 배포, 스케줄링, API 요청 등을 관리합니다.

**12. kube-apiserver의 역할은 무엇인가요?**
- **정답:** kubectl과 다른 Kubernetes 구성요소의 요청을 받는 중앙 API 창구입니다.

**13. etcd란 무엇인가요?**
- **정답:** Kubernetes Cluster의 상태와 설정 정보를 저장하는 Key-Value 데이터베이스입니다.

**14. kube-scheduler의 역할은 무엇인가요?**
- **정답:** 새 Pod를 어떤 Node에서 실행할지 결정합니다.

**15. kube-controller-manager의 역할은 무엇인가요?**
- **정답:** 현재 상태와 원하는 상태를 비교하고 원하는 상태가 유지되도록 조정합니다.

**16. Kubernetes에서 Container를 실제로 실행하는 Runtime으로 우리는 무엇을 사용했나요?**
- **정답:** containerd를 사용했습니다.

**17. kubelet과 containerd의 관계는 무엇인가요?**
- **정답:** kubelet이 Node에서 Pod 실행을 관리하고 containerd가 실제 Container를 실행합니다.

**18. Flannel은 무엇인가요?**
- **정답:** Kubernetes Pod Network를 구성하는 CNI Plugin입니다.

**19. CNI란 무엇인가요?**
- **정답:** Kubernetes에서 Pod Network를 구성하기 위한 네트워크 인터페이스 규격과 Plugin 체계입니다.

**20. Flannel을 설치하기 전에 Node가 NotReady였던 이유 중 하나는 무엇인가요?**
- **정답:** Pod Network가 아직 구성되지 않았기 때문입니다.

**21. 우리가 설정한 Pod Network 대역은 무엇이었나요?**
- **정답:** 10.244.0.0/16이었습니다.

**22. Node Network와 Pod Network는 같은 대역을 사용하나요?**
- **정답:** 아닙니다. 우리 환경에서는 Node는 192.168.111.x 대역, Pod는 10.244.x.x 대역을 사용했습니다.

**23. CoreDNS의 역할은 무엇인가요?**
- **정답:** Kubernetes 내부에서 이름 기반 통신을 가능하게 하는 DNS 역할을 합니다.

**24. kube-proxy의 역할은 무엇인가요?**
- **정답:** Service Traffic이 적절한 Pod로 전달될 수 있도록 각 Node의 네트워크 규칙을 관리합니다.

**25. DaemonSet이란 무엇인가요?**
- **정답:** 특정 Pod를 각 Node마다 하나씩 실행하도록 하는 Kubernetes 오브젝트입니다.

**26. kube-flannel과 kube-proxy가 각 Node마다 하나씩 실행된 이유는 무엇인가요?**
- **정답:** DaemonSet 방식으로 배포되기 때문입니다.

**27. kubeadm init은 무엇을 하나요?**
- **정답:** Control Plane을 초기화하고 Kubernetes Cluster를 생성합니다.

**28. kubeadm join은 무엇을 하나요?**
- **정답:** Worker Node를 기존 Kubernetes Cluster에 참여시킵니다.

**29. 6443 포트는 주로 어떤 구성요소와 관련되나요?**
- **정답:** Kubernetes API Server와 관련된 포트입니다.

**30. crictl은 어떤 용도로 사용했나요?**
- **정답:** containerd 같은 CRI Runtime의 Container 상태와 로그를 직접 확인하는 데 사용했습니다.

**31. kops란 무엇인가요?**
- **정답:** AWS 같은 Cloud에서 인프라 생성과 Kubernetes Cluster 구축을 더 넓게 자동화하는 도구입니다.

**32. kubeadm과 kops의 차이는 무엇인가요?**
- **정답:** kubeadm은 준비된 서버에 Kubernetes를 구축하고, kops는 Cloud 인프라 생성까지 더 넓게 자동화합니다.

**33. kops의 State Store로 AWS에서 사용할 수 있는 대표적인 서비스는 무엇인가요?**
- **정답:** Amazon S3입니다.

**34. GKE란 무엇인가요?**
- **정답:** Google이 제공하는 Managed Kubernetes 서비스인 Google Kubernetes Engine입니다.

**35. kubeadm과 GKE의 가장 큰 차이는 무엇인가요?**
- **정답:** kubeadm은 사용자가 Cluster를 직접 구축하고 관리하지만, GKE는 Google이 Control Plane 등 많은 관리 작업을 대신합니다.

**36. AWS와 Azure의 Managed Kubernetes 서비스는 각각 무엇인가요?**
- **정답:** AWS는 EKS, Azure는 AKS입니다.

**37. Minikube, kubeadm, kops, GKE를 한 줄씩 구분하면?**
- **정답:**
  - Minikube = 로컬 학습용
  - kubeadm = 직접 Cluster 구축
  - kops = Cloud 인프라 + Kubernetes 구축 자동화
  - GKE = Google Managed Kubernetes

**38. 5장에서 가장 중요한 실습 경험은 무엇이었나요?**
- **정답:** kubeadm으로 Control Plane 1대와 Worker 2대의 3 Node Kubernetes Cluster를 직접 구축하고 모두 Ready 상태까지 만든 것입니다.




# Kubernetes 5장 - Kubernetes 구조와 클러스터 구성

## 1. Kubernetes란?

Kubernetes는 여러 서버(Node)에서 여러 Container를 자동으로 배치하고 운영하기 위한 Container Orchestration 플랫폼이다.

Docker만 사용하는 경우 서버에서 Container를 직접 실행하고 관리해야 하지만, Kubernetes를 사용하면 다음과 같은 작업을 자동화할 수 있다.

- Container 배치
- Container 장애 복구
- Pod 개수 유지
- 여러 Node에 워크로드 분산
- 애플리케이션 배포 및 확장
- Service Discovery
- Load Balancing

기본적인 구조는 다음과 같다.

```text
Kubernetes Cluster

├─ Control Plane
│
└─ Worker Node
```

Control Plane은 Kubernetes 전체를 관리하고,
Worker Node는 실제 애플리케이션 Pod를 실행한다.

---

# 2. Kubernetes 전체 구조

```text
                         Kubernetes Cluster

┌──────────────────────────────────────────────────────┐
│                                                      │
│                    Control Plane                     │
│                                                      │
│ kubectl ───────→ API Server ←──── Controller Manager │
│                     ↑  ↓      ←──── Scheduler        │
│                     │ etcd                           │
│                     │                                │
│─────────────────────┼────────────────────────────────│
│                     │                                │
│                  Worker Nodes                        │
│                                                      │
│     Worker1                        Worker2            │
│     ├─ kubelet                     ├─ kubelet         │
│     ├─ kube-proxy                  ├─ kube-proxy      │
│     ├─ containerd                  ├─ containerd      │
│     └─ Pod                         └─ Pod             │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

# 3. Kubernetes 주요 구성요소

## 3.1 kubectl

`kubectl`은 사용자가 Kubernetes에 명령을 전달하기 위한 CLI 도구이다.

예:

```bash
kubectl get pods
kubectl get nodes
kubectl apply -f deployment.yaml
```

`kubectl`이 Worker Node에 직접 명령하는 것이 아니다.

```text
사용자
 ↓
kubectl
 ↓
API Server
```

즉 `kubectl`은 Kubernetes API Server에 요청을 보낸다.

---

## 3.2 API Server

API Server는 Kubernetes의 중앙 통신 창구이다.

```text
kubectl
Controller
Scheduler
kubelet
Ingress Controller

      ↕
  API Server
```

예를 들어:

```bash
kubectl apply -f deployment.yaml
```

을 실행하면:

```text
kubectl
 ↓
API Server
 ↓
Kubernetes에 원하는 상태 등록
```

과정이 진행된다.

현재 실습 환경의 API Server:

```text
https://192.168.111.137:6443
```

즉:

```text
kubectl
 ↓
192.168.111.137:6443
 ↓
API Server
```

구조이다.

---

## 3.3 etcd

`etcd`는 Kubernetes Cluster의 상태를 저장하는 Key-Value 데이터 저장소이다.

예:

```text
Node 정보
Pod 정보
Deployment 정보
Service 정보
ConfigMap
Secret
Ingress
```

단순히 사용자가 실행한 명령을 기록하는 곳이라기보다는,

```text
"Kubernetes Cluster가 어떤 상태여야 하는가?"
```

와 관련된 상태 정보를 저장한다.

구조:

```text
Kubernetes Component
       ↓
   API Server
       ↓
      etcd
```

Controller나 Scheduler가 일반적으로 etcd에 직접 접근하는 것이 아니라 API Server를 통해 상태를 확인한다.

---

## 3.4 Controller Manager

Controller는 Kubernetes의 **현재 상태와 원하는 상태를 비교하고 두 상태를 맞추는 역할**을 한다.

예를 들어:

```yaml
replicas: 3
```

이라고 설정했다고 가정한다.

원하는 상태:

```text
Pod = 3개
```

현재 상태:

```text
Pod = 2개
```

Controller는:

```text
원하는 상태 = 3
현재 상태 = 2

        ↓

Pod 1개 부족

        ↓

Pod 추가 생성
```

을 통해 원하는 상태로 맞춘다.

핵심:

```text
원하는 상태
     ↓
현재 상태와 비교
     ↓
다르면 맞춘다
```

이것이 Kubernetes의 핵심 개념인 **Desired State**이다.

---

## 3.5 Scheduler

Scheduler는 새롭게 생성될 Pod가 어느 Worker Node에서 실행될지 결정한다.

```text
새 Pod 생성 필요

      ↓

Scheduler

      ↓

worker1?
worker2?

      ↓

적절한 Node 선택
```

중요:

```text
Scheduler
= Pod를 직접 실행하지 않음

Scheduler
= Pod가 실행될 Node를 결정
```

---

## 3.6 kubelet

`kubelet`은 각 Node에서 실행되는 Kubernetes Agent이다.

Scheduler가 Pod를 특정 Node에 배치하면,
해당 Node의 kubelet이 실제 Pod 실행을 관리한다.

```text
Scheduler
 ↓
worker2 선택
 ↓
worker2의 kubelet
 ↓
Container Runtime
 ↓
Pod 실행
```

중요한 차이:

```text
kubectl
= 사용자 명령을 API Server에 전달

kubelet
= Worker Node에서 실제 Pod 실행을 관리
```

---

## 3.7 Container Runtime

현재 실습 환경에서는 `containerd`를 사용한다.

```text
kubelet
 ↓
containerd
 ↓
Container
 ↓
Pod 실행
```

실제 Node 정보에서도 확인할 수 있다.

```text
CONTAINER-RUNTIME
containerd://2.2.x
```

---

## 3.8 kube-proxy

`kube-proxy`는 Kubernetes Service의 네트워크 트래픽 처리에 관여한다.

현재 Cluster에서는 각 Node에 하나씩 실행되고 있다.

```text
control-plane
└─ kube-proxy

worker1
└─ kube-proxy

worker2
└─ kube-proxy
```

이후 13장에서 배우는 `DaemonSet`과 연결되는 구조이다.

---

## 3.9 CoreDNS

CoreDNS는 Kubernetes 내부 DNS 역할을 한다.

예를 들어 WEB Pod가 WAS Service를 다음 이름으로 호출할 수 있다.

```text
http://was-service
```

구조:

```text
WEB Pod
 ↓
was-service
 ↓
WAS Pod
```

Pod IP를 직접 사용하는 대신 Service 이름을 통해 접근할 수 있도록 DNS 기능을 제공한다.

---

# 4. kubectl apply 이후 Pod가 실행되기까지

사용자가 다음 명령을 실행한다고 가정한다.

```bash
kubectl apply -f deployment.yaml
```

전체 흐름:

```text
사용자
 ↓
kubectl
 ↓
API Server
 ↓
원하는 상태 저장
 ↓
Controller가 상태 확인
 ↓
Pod 생성 필요 확인
 ↓
Scheduler가 실행할 Node 결정
 ↓
해당 Worker Node의 kubelet
 ↓
Container Runtime(containerd)
 ↓
Pod 실행
```

한 줄로 요약하면:

```text
kubectl
 ↓
API Server
 ↓
Controller
 ↓
Scheduler
 ↓
kubelet
 ↓
Container Runtime
 ↓
Pod
```

API Server 뒤에는:

```text
API Server
 ↓
etcd
```

가 있어 Kubernetes Cluster 상태를 저장한다.

---

# 5. 실습 환경

현재 Kubernetes Cluster:

```text
Control Plane
IP: 192.168.111.137

Worker1
IP: 192.168.111.138

Worker2
IP: 192.168.111.134
```

Kubernetes Version:

```text
v1.37.1
```

Container Runtime:

```text
containerd
```

Pod Network:

```text
10.244.0.0/16
```

---

# 6. Node 상태 확인

명령어:

```bash
kubectl get nodes -o wide
```

실습 결과:

```text
NAME            STATUS   ROLES           INTERNAL-IP
control-plane   Ready    control-plane   192.168.111.137
worker1         Ready    <none>          192.168.111.138
worker2         Ready    <none>          192.168.111.134
```

확인 포인트:

```text
STATUS = Ready
```

이면 Kubernetes가 해당 Node를 정상적으로 사용할 수 있다는 의미이다.

또한 출력에서 Container Runtime을 확인할 수 있다.

```text
containerd://2.2.x
```

---

# 7. Control Plane 구성요소 확인

명령어:

```bash
kubectl get pods -n kube-system -o wide
```

주요 구성요소:

```text
etcd-control-plane

kube-apiserver-control-plane

kube-controller-manager-control-plane

kube-scheduler-control-plane
```

즉 실제 Control Plane에는 다음 구성요소가 실행되고 있다.

```text
Control Plane

├─ API Server
├─ etcd
├─ Controller Manager
└─ Scheduler
```

---

# 8. CoreDNS 확인

명령어:

```bash
kubectl get pods -n kube-system -o wide
```

실습 환경에서는 다음과 같은 CoreDNS Pod를 확인할 수 있었다.

```text
coredns-xxxxx
coredns-xxxxx
```

역할:

```text
CoreDNS
= Kubernetes 내부 DNS
```

예:

```text
WEB Pod
 ↓
was-service
 ↓
CoreDNS가 Service 이름 해석
 ↓
WAS Service
 ↓
WAS Pod
```

---

# 9. kube-proxy 확인

명령어:

```bash
kubectl get pods -n kube-system -o wide
```

현재 환경에서는:

```text
kube-proxy → control-plane
kube-proxy → worker1
kube-proxy → worker2
```

처럼 Node마다 하나씩 실행되고 있었다.

```text
control-plane
└─ kube-proxy

worker1
└─ kube-proxy

worker2
└─ kube-proxy
```

---

# 10. Kubernetes Cluster 정보 확인

명령어:

```bash
kubectl cluster-info
```

실습 결과:

```text
Kubernetes control plane is running at
https://192.168.111.137:6443
```

따라서:

```text
kubectl
 ↓
192.168.111.137:6443
 ↓
API Server
```

구조로 요청이 전달된다.

---

# 11. API Server Readiness 확인

명령어:

```bash
kubectl get --raw='/readyz'
```

정상 결과:

```text
ok
```

의미:

```text
API Server가 요청을 받을 준비가 되어 있음
```

잘못된 예:

```bash
kubectl get --raw= '/readyz'
```

`=` 뒤에 공백이 들어가면 명령어가 정상적으로 처리되지 않는다.

정상:

```bash
kubectl get --raw='/readyz'
```

---

# 12. kubelet 상태 확인

각 Node에서 다음 명령어를 실행한다.

```bash
systemctl is-active kubelet
```

정상:

```text
active
```

자세한 확인:

```bash
systemctl status kubelet --no-pager
```

Worker1 접속:

```bash
ssh ubuntu@192.168.111.138
```

확인:

```bash
systemctl is-active kubelet
```

Worker2 접속:

```bash
ssh ubuntu@192.168.111.134
```

확인:

```bash
systemctl is-active kubelet
```

세 Node 모두:

```text
active
```

상태임을 확인했다.

---

# 13. Pod가 어느 Node에서 실행되는지 확인

명령어:

```bash
kubectl get pods -A -o wide
```

확인 항목:

```text
NAME
IP
NODE
STATUS
```

예:

```text
Pod A
IP: 10.244.1.x
NODE: worker1

Pod B
IP: 10.244.2.x
NODE: worker2
```

이 결과를 통해:

```text
Pod 생성 필요
 ↓
Scheduler
 ↓
Node 선택
 ↓
해당 Node의 kubelet
 ↓
Pod 실행
```

구조를 확인할 수 있다.

---

# 14. kubeadm 환경의 Static Pod

현재 Cluster는 `kubeadm`으로 구축했다.

kubeadm 환경에서는 다음 Control Plane 구성요소들이 일반적으로 Static Pod 형태로 실행된다.

```text
kube-apiserver
etcd
kube-controller-manager
kube-scheduler
```

Manifest 확인:

```bash
sudo ls -l /etc/kubernetes/manifests/
```

예:

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

구조:

```text
/etc/kubernetes/manifests/
          ↓
Control Plane의 kubelet
          ↓
Static Pod 실행
```

즉 Control Plane 구성요소 역시 실제 실행되는 프로세스이며,
kubeadm 환경에서는 kubelet이 Static Pod 형태로 실행시킨다.

---

# 15. Kubernetes 구조 최종 정리

```text
Kubernetes Cluster

├─ Control Plane
│
│  ├─ API Server
│  │   └─ Kubernetes 중앙 통신 창구
│  │
│  ├─ etcd
│  │   └─ Cluster 상태 저장
│  │
│  ├─ Scheduler
│  │   └─ Pod가 실행될 Node 결정
│  │
│  └─ Controller Manager
│      └─ 원하는 상태와 현재 상태를 맞춤
│
├─ Worker1
│  ├─ kubelet
│  ├─ kube-proxy
│  ├─ containerd
│  └─ Pod
│
└─ Worker2
   ├─ kubelet
   ├─ kube-proxy
   ├─ containerd
   └─ Pod
```

---

# 16. 구성요소 한 줄 정리

```text
kubectl
= Kubernetes에 명령을 보내는 CLI

API Server
= Kubernetes의 중앙 통신 창구

etcd
= Kubernetes Cluster 상태 저장소

Controller
= 현재 상태를 원하는 상태에 맞춤

Scheduler
= Pod가 실행될 Node를 선택

kubelet
= 해당 Node에서 Pod 실행을 관리

containerd
= 실제 Container 실행

kube-proxy
= Service 네트워크 트래픽 처리

CoreDNS
= Kubernetes 내부 DNS
```

---

# 17. 가장 중요한 실행 흐름

```text
사용자
 ↓
kubectl
 ↓
API Server
 ↓
Controller
 ↓
Scheduler
 ↓
Worker Node
 ↓
kubelet
 ↓
containerd
 ↓
Pod
```

상태 저장:

```text
API Server
 ↓
etcd
```

---

# 18. 중요한 구분

## kubectl과 kubelet

```text
kubectl
= 사용자가 Kubernetes에 명령을 보내는 프로그램

kubelet
= 각 Node에서 실제 Pod 실행을 관리하는 프로그램
```

## Scheduler와 kubelet

```text
Scheduler
= 어느 Node?

kubelet
= 그 Node에서 실제 실행
```

## Controller와 Scheduler

```text
Controller
= Pod가 필요한가?

Scheduler
= 그 Pod를 어느 Node에 둘까?
```

## API Server와 etcd

```text
API Server
= Kubernetes 중앙 창구

etcd
= Kubernetes 상태 저장소
```

---

# 19. 5장 핵심 질문과 답

## Q1. kubectl은 Worker Node에 직접 명령하는가?

아니다.

```text
kubectl
 ↓
API Server
```

로 요청한다.

---

## Q2. etcd는 무엇인가?

Kubernetes Cluster의 상태 정보를 저장하는 데이터 저장소이다.

---

## Q3. 원하는 Pod가 3개인데 현재 2개라면?

Controller가 원하는 상태와 현재 상태의 차이를 확인하고 다시 원하는 상태로 맞춘다.

---

## Q4. Pod가 어느 Node에서 실행될지 누가 결정하는가?

```text
Scheduler
```

---

## Q5. Worker Node에서 실제 Pod 실행을 관리하는 것은?

```text
kubelet
```

---

## Q6. Container를 실제 실행하는 Runtime은?

현재 환경에서는:

```text
containerd
```

를 사용한다.

---

## Q7. API Server가 중요한 이유는?

Kubernetes 구성요소들이 Cluster 상태를 조회하고 변경하기 위해 사용하는 중앙 통신 창구이기 때문이다.

---

## Q8. Controller가 etcd를 직접 읽는가?

일반적으로 직접 접근하지 않는다.

```text
Controller
 ↓
API Server
 ↓
etcd
```

구조로 이해한다.

---

## Q9. Scheduler가 Pod를 직접 실행하는가?

아니다.

```text
Scheduler
= Node 결정

kubelet
= 해당 Node에서 Pod 실행 관리
```

---

# 20. 실습 명령어 모음

## Node 확인

```bash
kubectl get nodes
```

```bash
kubectl get nodes -o wide
```

---

## Kubernetes System Pod 확인

```bash
kubectl get pods -n kube-system
```

```bash
kubectl get pods -n kube-system -o wide
```

---

## 모든 Namespace의 Pod 확인

```bash
kubectl get pods -A
```

```bash
kubectl get pods -A -o wide
```

---

## Cluster 정보 확인

```bash
kubectl cluster-info
```

---

## API Server Ready 확인

```bash
kubectl get --raw='/readyz'
```

---

## kubelet 상태 확인

```bash
systemctl is-active kubelet
```

```bash
systemctl status kubelet --no-pager
```

---

## Static Pod Manifest 확인

```bash
sudo ls -l /etc/kubernetes/manifests/
```

---

# 21. 5장 구조 요약 이미지

이미지 파일을 Repository에 다음과 같이 저장한다.

```text
docs/
└─ images/
   └─ kubernetes-ch05-summary.png
```

README에 다음 내용을 작성하면 이미지가 표시된다.

```markdown
![Kubernetes 5장 구조 요약](./docs/images/kubernetes-ch05-summary.png)
```

---

# 22. 5장 최종 요약

Kubernetes Cluster는 크게 `Control Plane`과 `Worker Node`로 구성된다.

```text
Control Plane
= Kubernetes 전체 관리

Worker Node
= 실제 Pod 실행
```

사용자가 `kubectl`을 사용해 명령하면:

```text
kubectl
 ↓
API Server
```

로 요청이 전달된다.

Kubernetes는 원하는 상태를 저장하고 Controller가 현재 상태와 비교한다.

Pod가 필요하면:

```text
Controller
 ↓
Scheduler
 ↓
Node 선택
 ↓
kubelet
 ↓
containerd
 ↓
Pod
```

순서로 실제 Pod가 실행된다.

핵심적으로 기억해야 할 구조는 다음과 같다.

```text
kubectl
 ↓
API Server
 ↓
Controller
 ↓
Scheduler
 ↓
kubelet
 ↓
Container Runtime
 ↓
Pod
```

그리고 Kubernetes Cluster 상태는:

```text
API Server
 ↓
etcd
```

에 저장된다.

---

# 23. 5장 학습 완료 체크

- [x] Kubernetes Cluster 구조 이해
- [x] Control Plane과 Worker Node 차이 이해
- [x] API Server 역할 이해
- [x] etcd 역할 이해
- [x] Controller 역할 이해
- [x] Scheduler 역할 이해
- [x] kubelet 역할 이해
- [x] containerd 확인
- [x] kube-proxy 확인
- [x] CoreDNS 확인
- [x] Node 상태 확인
- [x] API Server 주소 확인
- [x] API Server Ready 확인
- [x] kubelet 상태 확인
- [x] Static Pod 구조 확인
- [x] kubectl → API Server → Scheduler → kubelet → Pod 흐름 이해



<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/e358f63e-a727-44ce-8364-ce4e369abe83" />















</details>
<details>
<summary><b>6장 쿠버네티스 시작하기</b></summary>

## Kubernetes는 Container를 직접 관리하지 않는다.
1. Docker는 Container를 직접 관리했다.
2. Kubernetes는 기본적으로 Pod가 Container를 관리한다.
3. Kubernetes가 직접 다루는 기본 단위는 Container가 아니라 Pod이다.
4. Pod : Container 실행 기본 단위
5. ReplicaSet : Pod 개수 유지
6. Deployment : 배포 / 업데이트 관리
7. Service : Pod로 가는 안정적인 접근 지점
8. Deployment → ReplicaSet → Pod → Container

## Pod란?
1. Kubernetes에서 가장 기본이 되는 실행 단위가 Pod이다.
2. Pod는 Kubernetes가 Container를 실행하고 관리하기 위해 사용하는 기본 단위
3. Container는 실제 애플리케이션 프로세스가 실행되는 곳이고, Pod는 그 Container를 감싸는 Kubernetes 단위
4. Pod (Contaienr, Network환경, Pod IP, Volume 등을 연결할 수 있는 실행 환경)
5. 보통 Pod 하나에 Container 하나
6. 필요하다면 여려 Container도 가능하다. 보조 Container 패턴을 Sidecar라고 부른다.

## Pod에는 IP가 생긴다.
1. Pod가 만들어지면 Kubernetes 네트워크에서 사용할 Pod IP를 받을 수 있다.
2. Flannel/CNI → Pod Network 구성 → 각 Pod에 IP 부여 → Pod끼리 통신 가능

## Pod는 특정 Node에서 실행된다.
1. Pod를 만들면 Kubernetes가 Pod를 실행할 Node를 결정한다.
2. 어느 Node에 Pod를 놓을지 결정하는 구성요소가 Scheduler이다.
3. 새 pod 필요 → Scheduler → 어느 Node가 적절할까? → worker1 선택 → Pod 실행
4. 선택된 Node에서는 kubelet → containerd → Container 실행
5. Pod 생성 요청 → API Server → Scheduler 가 Node 선택 → 해당 Node의 kubelet → containerd → Container 실행

## Pod는 영구적인 서버가 아니다.
1. Pod는 필요에 따라 생성, 삭제, 재생성, 교체가 될 수 있다.
2. 재생성되면 IP도 바뀔 수 있다.

## Pod 사용하기
1. kubectl run nginx-pod --image=nginx
2. kubectl run : Pod 생성
3. nginx-pod : Pod 이름
4. --image=nginx : nginx Image 사용

## Pod 확인
1. kubectl get pods
2. kubectl get pods -o wide

## pod 상세 정보
1. kubectl describe pod nginx-pod
2. Node, Pod IP, Image, Container 상태, Conditions, Events 등을 볼 수 있다.

## Pod 로그 확인
1. kubectl logs nginx-pod

## Pod 내부 접속
1. kubectl exec -it nginx-pod -- /bin/bash

## Pod 삭제
1. kubectl delete pod nginx-pod

## Pod vs Docker Container
1. Docker는 Container를 직접 다루고, Kubernetes는 Pod를 통해 Container를 다룬다.

## Container 위에 Pod라는 단위가 필요한 이유
1. Kubernetes는 경우에 따라 서로 아주 밀접한 여러 Container를 하나의 실행 단위로 묶을 필요가 있다.
2. Pod (Main APP Container + Helper Container)
3. 이 두 Container는 같은 Node에 배치되고, 같은 Pod 생명주기를 가지고, 같은 네트워크 공간을 공유하고, Volume도 함께 사용할 수 있다.

## IP도 Pod 기준
1. 같은 Pod 안에 여러 Container가 있어도 같은 Pod 네트워크 공간을 공유한다.

## Pod가 Node에 배치된다.
1. Kubernetes Scheduler가 배치하는 대상도 Cotainer가 아니라 Pod이다.

## 완전한 애플리케이션으로서의 Pod
1. 서로 아주 밀접하게 협력해야 하는 Container들을 하나의 Pod 안에 묶을 수 있다.
2. Helper Container가 App Container에 접근하려면 curl localhost:8080 처럼 할 수 있다.
3. 서로 독립적으로 운영해야 하는 애플리케이션은 Pod를 부닐하고, 반드시 함께 움직여야 하는 보조 Container만 같은 Pod에 넣는다.
4. 이런 보조 Container를 Sidecar Container라고 한다.

## ReplicaSet이란 
1. ReplicaSet은 원하는 개수의 Pod가 항상 유지되도록 관리하는 Controller이다.

## ReplicaSet을 사용하는 이유
1. Pod가 삭제되거나 장애가 발생해도 원하는 개수를 자동으로 유지하기 위해서

## ReplicaSet은 어떤 Pod를 관리할까
1. Label을 보고 자기 관리 대상 Pod를 찾는다.

## Label이란?
1. Label은 Kubernetes 오브젝트에 붙이는 이름표/태그라고 생각하면 된다.

## Selector란?
1. 어떤 Label을 가진 Pod를 관리할지 선택하는 조건
2. Label : Pod에 붙은 이름표
3. Selector : 그 이름표를 보고 Pod 선택

## Controller
1. ReplicaSet은 Controller이다.
2. 현재 상태를 계속 확인하면서 원하는 상태로 맞추는 역할이다.
3. ReplicaSet은 Label과 Selector로 관리할 Pod를 찾고, replicas 값만큼 Pod 개수를 계속 유지한다.










## ReplicaSet은 어떻게 동작하나 
1. ReplicaSet의 핵심은 계속 원하는 상태와 현재 상태를 비교하는 것이다.
2. Controller = 현재 상태를 감시하고 원하는 상태로 계속 맞추는 역할
3. RaplicaSet은 Selector로 관리할 Pod를 찾고, 현재 개수와 replicas 값을 비교해 부족하면 template으로 새 Pod를 만들어 원하는 개수를 유지한다.

## 디플로이먼트 : ReplicaSet과 Pod의 배포를 관리
1. Deployment는 ReplicaSet보다 한 단계 위에서 배포 전체를 관리하는 오브젝트입니다.
2. ReplicaSet은 주로 Pod 개수 유지가 핵심
3. Deployment는 Pod 개수 유지 + 버전 업데이트 + Rolling Update + Rollback 까지 관리한다.

## ReplicaSet과 Deployment 차이
1. ReplicaSet : Pod 개수 유지
2. Deployment : ReplicaSet 관리 + 애플리케이션 배포 관리
3. Deployment 는 사용자가 작성

## Deployment 사용하기
1. Deployment를 만들면 Kubernetes가 그 아래 ReplicaSet과 Pod를 자동으로 만든다.

## 구조
1. 사용자가 Deployment 생성
2. Deployment가 ReplicaSet 관리
3. ReplicaSet이 pod 개수 관리
4. Pod 안에서 Container 실행

## Deployment를 왜 쓰는가?
1. ReplicaSet은 Pod 개수를 원하는 만큼 유지하는 것이다.
2. 버전을 바꾸고 싶다면 Deployment는 기존 Pod를 조금씩 줄이고 새 버전 Pod를 조금씩 늘리는 방식으로 교체한다. 이것이 Rolling Update이다.
3. 서비스를 최대한 유지하면서 점진적으로 새 버전으로 교체 하는 것이다.

## 왜 ReplicaSet만으로는 부족한가.
1. ReplicaSet은 버전을 관리하는 용도가 핵심이 아니다.
2. Deployment는 ReplicaSet위에서 버전을 관리한다.

## Rolling Update
1. kubectl set image deployment/nginx-deployment nginx=nginx:1.27
2. deployment/nginx-deployment → nginx-deployment Deployment
3. nginx= → YAML에서 Container 이름이 nginx
4. nginx:1.27 → 변경할 새 이미지
5. nginx-deployment 안의 nginx Container 이미지를 nginx:1.27로 바꿔라 라는 뜻

## 업데이트 진행상황 확인
1. kubectl rollout status deployment/nginx-deployment
2. Deployment는 기존 ReplicaSet의 설정을 억지로 바꾸는 게 아니라, 새 버전용 ReplicaSet을 새로 만들고 교체한다.

## Rollout Histroy
1. kubectl rollout history deployment/nginx-deployment

## Rollback 
1. kubectl rollout undo deployment/nginx-deployment

## 그래서 Deployment를 왜 쓰느냐?
1. 일반 애플리케이션 배포는 ReplicaSet을 직접 만들기보다 Deployment를 주로 사용한다.
2. 운영에서는 단순히 Pod 수만 유지하는게 아니라 새 버전을 안전하게 배포하고 문제가 생기면 되돌리는 것이 중요하기 때문이다.

## Service의 종류
1. Pod 앞에 고정된 접근 지점을 만들어 주는 오브젝트이다.
2. Pod는 Ip가 바뀔 수 있지만 Service는 안정적인 주소를 제공한다.
3. Service의 종류는 크게 4개이다.

## ClusterIP
1. 가장 기본적인 Service 타입이다.
2. 클러스터 내부에서만 접근 가능하다.

## NodePort
1. 각 Node의 특정 포트를 열어서 외부에서 접근할 수 있게 하는 방식
2. 보통 NodePort 범위는 3000 ~ 32767이다.

## LoadBalancer
1. 클라우드의 Load Balancer와 연결해서 외부에 서비스를 노출하는 방식이다.

## ExternalName
1. Kubernetes 내부 이름을 외부 DNS 이름에 연결합니다.

## ClusterIP 타입의 Service
1. ClusterIP Service는 클러스터 내부에서 Pod들 앞에 고정된 주소를 제공한다.
2. Service는 고정된 접속 창구이다.

## Service YAML 만들기
1. nano service-clusterip.yaml
2.
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  type: ClusterIP

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
 3. nginx-service라는 Service를 만들고
 4. app=nginx라는 이름표를 가진 기존 Pod들을 찾아서 Service 80번 포트로 들어온 요청을 Pod 80번 포트로 보내라

 ## 제일 중요한 Selector
 1. selector:
       app: nginx
 2. Service가 app=nginx인 Pod를 찾아

## ClusterIP는 뭔가?
1. Service를 만들면 Kubernetes가 내부용 IP를 하나 준다.
2. 이게 ClusertIP이다.

## 실제 적용
1. kubectl apply -f service-clusterip.yaml
2. kubectl get svc

## 실제로 Service를 통해 접속해보기
1. kubectl run curl-test --image=curlimages/curl -it --rm -- sh
2. curl nginx-service
3. ClusterIP Service는 이미 존재하는 Pod들을 selector로 찾아서, Kubernetes 내부에서 안정적으로 접근할 수 있는 고정 창구를 만들어주는 기능이다.

## NodePort 타입의 Service
1. NodePort는 Kubernetes 바깥에서 Node의 IP와 특정 포트를 통해 Service에 접근하게 해주는 방식이다.

## ClusterIP와 NodePort 차이
1. ClusterIP는 Kubernetes 내부에서만 접근했다.
2. NodePort는 외부에서 들어올 수 있다.

## NodePort 이름 유래
1. Node의 Port를 하나 열기 때문이다.
2. NodePort의 포트 범위는 30000 ~ 32767 이다.

## YAML 만들기
1. nano service-nodeport.yaml
2. 
apiVersion: v1
kind: Service

metadata:
  name: nginx-nodeport

spec:
  type: NodePort

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080  

## port / targetPort / nodePort
1. nodePort : Node에서 외부에 열어 놓는 포트
2. port : Service의 포트
3. targerPort : 실제 Pod의 포트

## LoadBalancer Service
1. 클라우드의 로드밸런서를 Kubernetes Service와 연결하는 방식
2. 외부사용자는 로드밸런서 주소로 접속한다.

## NodePort와 차이
1. NodePort는 Node IP와 포트를 직접 알아야한다.
2. LoadBalancer는 알 필요가 없다.

## externalTrafficPolicy
1. 외부에서 들어온 요청을 어떤 Pod로 보낼까?
2. externalTrafficPolicy가 어떤 Pod까지 후보로 삼을지 결정한다.

## 기본값: Cluster
1. externalTrafficPolicy: Cluster
2. 요청이 들어온 Node에 Pod가 없어도, 클러스터 전체의 Pod 중 하나로 보내도 된다.
3. 클러스터 전체 Pod를 대상으로 트래픽을 분해 한다.

## Local
1. externalTrafficPolicy: Local
2. 요청이 들어온 바로 그 Node에 있는 pod에게만 전달한다.
3. 실제 사용자의 IP를 보존해야 할 때 사용한다.
4. Local은 아무 Node에게나 Pod가 있어도 되는 게 아니라, 외부 요청을 받을 Node에 실제 Pod가 있는지 중요하다.

## ExternalName
1. ExternalName은 앞에서 본 ClusterIP, NodePort, LoadBalancer와 성격이 다르다.
2. Kubernetes 내부 이름을 외부 DNS 이름에 연결하는 Service이다.
3. 쉽게 말하면 별명이다.
4. 외부 서비스를 Kubernetes 내부 Service 이름처럼 사용하고 싶을 때 쓴다.

## 확인문제

[6장 쿠버네티스 시작하기 - 확인문제 + 정답]

1. Kubernetes에서 Container를 직접 실행 단위로 다루기보다 사용하는 기본 단위는 무엇인가요?
정답: Pod

2. Pod와 Container는 같은 것인가요?
정답: 아닙니다. Container는 실제 애플리케이션 프로세스가 실행되는 단위이고, Pod는 하나 이상의 Container를 묶어 Kubernetes가 배치하고 관리하는 기본 단위입니다.

3. 일반적인 애플리케이션에서는 Pod 하나에 Container를 몇 개 두는 경우가 많나요?
정답: 보통 1개

4. Pod 하나 안에 여러 Container를 넣을 수 있나요?
정답: 네. 서로 밀접하게 협력해야 하는 Container들을 하나의 Pod에 넣을 수 있습니다.

5. 같은 Pod 안의 Container들은 네트워크를 공유하나요?
정답: 네. 같은 네트워크 공간을 공유합니다.

6. 같은 Pod 안의 Container끼리는 localhost로 통신할 수 있나요?
정답: 네.

7. 같은 Pod 안의 Container들이 서로 다른 Node에 배치될 수 있나요?
정답: 아닙니다. 같은 Pod의 Container들은 같은 Node에 배치됩니다.

8. Pod를 어느 Node에 배치할지 결정하는 Kubernetes 구성요소는 무엇인가요?
정답: Scheduler

9. Node에서 Pod 실행을 관리하는 구성요소는 무엇인가요?
정답: kubelet

10. Pod를 직접 생성한 뒤 삭제하면 자동으로 다시 만들어지나요?
정답: 아닙니다. ReplicaSet이나 Deployment 같은 Controller가 관리하지 않는 Pod는 삭제되면 그대로 사라집니다.

11. ReplicaSet의 핵심 역할은 무엇인가요?
정답: 원하는 개수의 Pod를 항상 유지하는 것입니다.

12. replicas: 3의 의미는 무엇인가요?
정답: Pod를 3개 유지하겠다는 의미입니다.

13. ReplicaSet은 어떤 Pod를 자신이 관리할 Pod인지 어떻게 찾나요?
정답: Label과 Selector를 이용합니다.

14. Label이란 무엇인가요?
정답: Pod 등의 Kubernetes 오브젝트에 붙이는 이름표 또는 태그입니다.

15. Selector란 무엇인가요?
정답: 특정 Label을 가진 오브젝트를 선택하기 위한 조건입니다.

16. ReplicaSet의 template은 어떤 역할을 하나요?
정답: Pod가 부족할 때 새로 만들 Pod의 설계도를 정의합니다.

17. ReplicaSet에서 원하는 Pod 수가 5개인데 현재 4개라면 어떻게 되나요?
정답: ReplicaSet이 Pod 하나를 새로 생성해서 다시 5개로 맞춥니다.

18. ReplicaSet에서 원하는 Pod 수가 3개인데 현재 4개라면 어떻게 되나요?
정답: Pod 하나를 제거해서 다시 3개로 맞춥니다.

19. Kubernetes Controller의 기본 동작 방식은 무엇인가요?
정답: Desired State와 Current State를 비교해서 현재 상태를 원하는 상태로 맞춥니다.

20. Deployment의 기본 구조를 순서대로 말하면 어떻게 되나요?
정답: Deployment → ReplicaSet → Pod → Container

21. Deployment를 사용하는 주된 이유는 무엇인가요?
정답: ReplicaSet과 Pod의 배포를 관리하고 Rolling Update와 Rollback 같은 배포 기능을 사용하기 위해서입니다.

22. 일반 애플리케이션 배포에서 ReplicaSet을 직접 만들기보다 주로 무엇을 사용하나요?
정답: Deployment

23. Deployment를 만들면 ReplicaSet도 직접 따로 만들어야 하나요?
정답: 아닙니다. Deployment가 ReplicaSet을 자동으로 생성하고 관리합니다.

24. Rolling Update란 무엇인가요?
정답: 기존 Pod를 한꺼번에 제거하지 않고 새 버전 Pod로 점진적으로 교체하는 방식입니다.

25. Rolling Update의 장점은 무엇인가요?
정답: 서비스를 최대한 유지하면서 애플리케이션 버전을 변경할 수 있습니다.

26. Deployment에서 이미지 버전을 변경하면 어떻게 되나요?
정답: 새로운 Pod Template에 해당하는 ReplicaSet이 만들어지고 새 ReplicaSet으로 점진적으로 전환됩니다.

27. kubectl rollout status deployment/nginx-deployment 명령은 무엇을 확인하나요?
정답: Deployment의 배포 진행 상태를 확인합니다.

28. kubectl rollout history deployment/nginx-deployment 명령은 무엇을 확인하나요?
정답: Deployment의 Rollout Revision 이력을 확인합니다.

29. kubectl rollout undo deployment/nginx-deployment 명령의 역할은 무엇인가요?
정답: 이전 배포 Revision으로 Rollback합니다.

30. Service를 사용하는 가장 중요한 이유는 무엇인가요?
정답: Pod가 재생성되면서 IP가 변경되어도 안정적인 접근 지점을 제공하기 위해서입니다.

31. Service가 Pod를 직접 생성하나요?
정답: 아닙니다. Service는 이미 존재하는 Pod를 찾아 연결합니다.

32. Deployment와 Service의 역할 차이는 무엇인가요?
정답: Deployment는 Pod를 생성, 유지, 배포하고 Service는 해당 Pod들에 접근할 수 있는 안정적인 창구를 제공합니다.

33. Service는 어떤 Pod와 연결할지 어떻게 결정하나요?
정답: Selector를 이용해 일치하는 Label을 가진 Pod를 찾습니다.

34. ClusterIP Service란 무엇인가요?
정답: Kubernetes 클러스터 내부에서 Pod에 접근할 수 있도록 하는 기본 Service 타입입니다.

35. ClusterIP의 대표적인 용도는 무엇인가요?
정답: Kubernetes 내부의 애플리케이션끼리 통신할 때 사용합니다.

36. Pod IP와 Service ClusterIP의 차이는 무엇인가요?
정답: Pod IP는 개별 Pod의 주소로 Pod 재생성 시 변경될 수 있지만 Service ClusterIP는 Service의 안정적인 내부 접근 주소입니다.

37. Service YAML에서 port는 무엇인가요?
정답: Service가 요청을 받는 포트입니다.

38. Service YAML에서 targetPort는 무엇인가요?
정답: 요청을 실제 Pod의 어느 포트로 전달할지 지정하는 값입니다.

39. port: 8080, targetPort: 80이면 어떤 의미인가요?
정답: Service의 8080번 포트로 들어온 요청을 Pod의 80번 포트로 전달한다는 뜻입니다.

40. NodePort Service란 무엇인가요?
정답: Node의 특정 포트를 외부에 열어서 Node IP + Port로 Service에 접근할 수 있게 하는 타입입니다.

41. NodePort의 기본 포트 범위는 무엇인가요?
정답: 일반적으로 30000~32767입니다.

42. nodePort: 30080의 의미는 무엇인가요?
정답: Node의 30080번 포트를 외부 접근용으로 사용한다는 의미입니다.

43. NodePort에서 요청 흐름을 간단히 표현하면 어떻게 되나요?
정답: 외부 사용자 → Node IP:NodePort → Service → Pod

44. 같은 Pod들을 바라보는 ClusterIP Service와 NodePort Service를 동시에 만들 수 있나요?
정답: 네. 같은 Label의 Pod를 대상으로 여러 Service를 만들 수 있습니다.

45. LoadBalancer Service란 무엇인가요?
정답: 클라우드의 Load Balancer와 연동해서 외부에서 Service에 접근할 수 있게 하는 타입입니다.

46. VMware + kubeadm 환경에서 type: LoadBalancer를 만들면 AWS Load Balancer 같은 것이 자동으로 생성되나요?
정답: 아닙니다. 연동된 Load Balancer 구현이 없다면 EXTERNAL-IP가 <pending> 상태가 될 수 있습니다.

47. ExternalName Service는 일반적인 Service처럼 Pod로 트래픽을 전달하나요?
정답: 아닙니다. Kubernetes 내부 이름을 외부 DNS 이름에 연결하는 방식입니다.

48. ExternalName의 예를 들면 어떻게 되나요?
정답: external-db라는 Service 이름을 db.example.com 같은 외부 DNS 이름에 연결할 수 있습니다.

49. externalTrafficPolicy: Cluster의 의미는 무엇인가요?
정답: 외부 요청을 클러스터 전체의 대상 Pod로 전달할 수 있다는 의미이며 기본값입니다.

50. externalTrafficPolicy: Local의 의미는 무엇인가요?
정답: 요청을 받은 Node에 존재하는 로컬 Pod로만 트래픽을 전달합니다.

51. externalTrafficPolicy: Local의 대표적인 장점은 무엇인가요?
정답: 원래 클라이언트 IP를 보존하는 데 유리합니다.

52. externalTrafficPolicy: Local의 단점은 무엇인가요?
정답: 요청을 받은 Node에 대상 Pod가 없으면 전달할 Pod가 없을 수 있고 Pod 배치에 따라 트래픽 분산이 불균형해질 수 있습니다.

53. kubectl get pods -o wide 명령으로 추가로 확인할 수 있는 것은 무엇인가요?
정답: Pod IP와 Pod가 배치된 Node 등을 확인할 수 있습니다.

54. kubectl describe pod <Pod이름>은 언제 사용하나요?
정답: Pod의 상세 상태, 설정, 이벤트 등을 확인할 때 사용합니다.

55. Pod 장애를 확인할 때 기본적인 확인 흐름은 어떻게 가져갈 수 있나요?
정답: kubectl get pods → kubectl describe pod → Events 확인 → kubectl logs 확인

56. kubectl logs <Pod이름>의 역할은 무엇인가요?
정답: Pod 내부 Container의 로그를 확인합니다.

57. kubectl exec -it <Pod이름> -- /bin/bash 또는 sh는 무엇을 위한 명령인가요?
정답: 실행 중인 Pod의 Container 내부로 들어가 명령을 실행하기 위한 명령입니다.

58. kubectl run curl-test --image=curlimages/curl -it --rm -- sh 명령의 목적은 무엇인가요?
정답: curl이 들어 있는 임시 테스트 Pod를 만들고 내부로 들어가 Service 통신을 확인한 뒤 종료하면 자동 삭제하기 위한 명령입니다.

59. ClusterIP 테스트를 위해 임시 Pod 안에서 curl nginx-service를 실행하는 이유는 무엇인가요?
정답: ClusterIP는 클러스터 내부용이므로 Kubernetes 내부 Pod에서 Service 이름을 통해 실제 nginx Pod까지 연결되는지 확인하기 위해서입니다.

60. 6장에서 가장 중요한 전체 구조는 무엇인가요?
정답:

Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container

Service
    ↓
Pod들에 안정적인 접근 제공


[6장 최종 핵심 정리]

Pod
= Container를 실행하는 Kubernetes 기본 단위

ReplicaSet
= 원하는 Pod 개수를 유지

Deployment
= ReplicaSet 관리 + 배포 + Rolling Update + Rollback

Service
= Pod에 접근하는 안정적인 창구

ClusterIP
= 클러스터 내부 접근

NodePort
= Node IP + Port로 외부 접근

LoadBalancer
= 클라우드 Load Balancer 연동

ExternalName
= Kubernetes 내부 이름을 외부 DNS 이름에 연결

전체 구조
Deployment → ReplicaSet → Pod → Container

Service → Pod


# Kubernetes 6장 - Pod, ReplicaSet, Deployment, Service

## 1. 6장 전체 핵심 구조

6장에서 가장 중요한 구조는 두 가지이다.

### 애플리케이션 관리 흐름

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↓
Container
```

### 트래픽 흐름

```text
Service
   ↓
Pod
```

두 구조는 Pod에서 만난다.

```text
                 Service
                    ↓
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        Pod1      Pod2      Pod3
          ↑         ↑         ↑
          └──── ReplicaSet ───┘
                    ↑
                Deployment
```

---

# 2. Pod

Pod는 Kubernetes에서 Container를 실행하는 기본 단위이다.

Docker에서는:

```text
Image
 ↓
Container
```

Kubernetes에서는:

```text
Pod
 ↓
Container
```

일반적으로 하나의 Pod에 하나의 주요 애플리케이션 Container를 실행한다.

```text
Pod
└─ nginx Container
```

필요하면 여러 Container를 하나의 Pod에 넣을 수도 있다.

```text
Pod
├─ Application Container
└─ Sidecar Container
```

같은 Pod의 Container들은 다음과 같은 특징이 있다.

- 같은 Node에서 실행
- 네트워크 공간 공유
- localhost 통신 가능
- Volume 공유 가능

---

# 3. ReplicaSet

ReplicaSet의 핵심 역할은:

```text
원하는 Pod 개수를 유지
```

하는 것이다.

예:

```yaml
replicas: 3
```

원하는 상태:

```text
Pod = 3개
```

현재 Pod 하나가 죽어서:

```text
Pod = 2개
```

가 되면 ReplicaSet이 다시 Pod를 생성한다.

```text
원하는 상태 = 3
현재 상태 = 2

      ↓

Pod 하나 추가 생성

      ↓

현재 상태 = 3
```

이것은 Kubernetes의 Desired State 개념과 연결된다.

---

# 4. Label과 Selector

ReplicaSet은 자신이 관리해야 하는 Pod를 Label과 Selector를 통해 찾는다.

Pod:

```text
label:
app=web
```

ReplicaSet:

```text
selector:
app=web
```

구조:

```text
ReplicaSet
selector: app=web
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
Pod   Pod   Pod
app=  app=  app=
web   web   web
```

---

# 5. Deployment

실제 애플리케이션 운영에서는 ReplicaSet을 직접 사용하는 것보다 Deployment를 주로 사용한다.

구조:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

Deployment의 역할:

```text
애플리케이션 배포
Pod 개수 관리
Rolling Update
Rollback
배포 버전 관리
```

정확히는 Deployment가 ReplicaSet을 관리하고,
ReplicaSet이 Pod 개수를 관리한다.

---

# 6. 직접 만드는 것과 자동 생성되는 것

일반적인 Deployment 사용 기준:

```text
직접 생성

Deployment
```

그러면 Kubernetes가:

```text
Deployment
   ↓
ReplicaSet 자동 생성
   ↓
Pod 자동 생성
```

한다.

Service는 Deployment가 자동으로 만드는 것이 아니다.

```text
Deployment
Service
```

는 서로 독립적인 Kubernetes Resource이다.

---

# 7. Deployment 생성 실습

실습 Namespace:

```bash
kubectl create namespace ch06-practice
```

Deployment 파일:

```bash
vi web-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: web
  namespace: ch06-practice

spec:
  replicas: 3

  selector:
    matchLabels:
      app: web

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
        - name: nginx
          image: nginx:1.26
          ports:
            - containerPort: 80
```

적용:

```bash
kubectl apply -f web-deployment.yaml
```

확인:

```bash
kubectl get deployment -n ch06-practice
```

```bash
kubectl get replicaset -n ch06-practice
```

```bash
kubectl get pods -n ch06-practice -o wide
```

한 번에 확인:

```bash
kubectl get deploy,rs,pod -n ch06-practice -o wide
```

구조:

```text
Deployment web
      ↓
ReplicaSet
      ↓
Pod 3개
```

---

# 8. Pod 자동 복구 실습

현재 Deployment:

```text
replicas = 3
```

Pod 확인:

```bash
kubectl get pods -n ch06-practice
```

Pod 하나 삭제:

```bash
kubectl delete pod <POD_NAME> -n ch06-practice
```

변화 확인:

```bash
kubectl get pods -n ch06-practice -w
```

Pod 하나가 삭제되면:

```text
원하는 상태 = 3
현재 상태 = 2
```

ReplicaSet이 새로운 Pod를 생성해서 다시 3개를 유지한다.

```text
Pod A
Pod B
Pod C

↓ Pod B 삭제

Pod A
Pod C

↓ ReplicaSet

Pod A
Pod C
Pod D
```

---

# 9. Pod 삭제와 replicas 변경의 차이

Pod 하나를 직접 삭제:

```text
원하는 상태 = 3
현재 상태 = 2

→ 다시 3개로 복구
```

반면 replicas 자체를 변경:

```text
replicas: 3
↓
replicas: 2
```

하면 원하는 상태 자체가 2개로 바뀐다.

```text
Pod 삭제
= 현재 상태가 원하는 상태와 달라짐

replicas 변경
= 원하는 상태 자체를 변경
```

---

# 10. Scale 실습

Pod 3개를 5개로 증가:

```bash
kubectl scale deployment web \
  --replicas=5 \
  -n ch06-practice
```

확인:

```bash
kubectl get deploy,rs,pod -n ch06-practice -o wide
```

정상:

```text
Deployment = 5/5
ReplicaSet DESIRED = 5
Pod = 5개
```

다시 3개:

```bash
kubectl scale deployment web \
  --replicas=3 \
  -n ch06-practice
```

---

# 11. 5장과 Deployment 생성 흐름 연결

사용자가:

```bash
kubectl apply -f web-deployment.yaml
```

을 실행하면 내부적으로 다음 흐름이 진행된다.

```text
kubectl
 ↓
API Server
 ↓
Deployment 등록
 ↓
Deployment Controller
 ↓
ReplicaSet
 ↓
ReplicaSet Controller
 ↓
Pod 필요
 ↓
Scheduler
 ↓
Node 결정
 ↓
해당 Node kubelet
 ↓
containerd
 ↓
Pod 실행
```

즉 5장에서 배운 Kubernetes 내부 구조와
6장의 Deployment가 연결된다.

---

# 12. Rolling Update

기존:

```text
nginx:1.26
```

새 버전:

```text
nginx:1.27
```

이미지 변경:

```bash
kubectl set image deployment/web \
  nginx=nginx:1.27 \
  -n ch06-practice
```

배포 상태 확인:

```bash
kubectl rollout status deployment/web -n ch06-practice
```

ReplicaSet 확인:

```bash
kubectl get rs -n ch06-practice
```

Rolling Update를 하면 새로운 ReplicaSet이 생성된다.

```text
Deployment
   │
   ├─ Old ReplicaSet
   │      ↓
   │   nginx:1.26
   │
   └─ New ReplicaSet
          ↓
       nginx:1.27
```

새로운 Pod를 점진적으로 늘리고 기존 Pod를 줄인다.

```text
기존 Pod 일부 유지
 ↓
새 Pod 생성
 ↓
새 Pod 정상 확인
 ↓
기존 Pod 제거
 ↓
반복
```

이 방식으로 서비스 중단을 줄인다.

---

# 13. Rollout 명령

배포 상태:

```bash
kubectl rollout status deployment/web -n ch06-practice
```

배포 이력:

```bash
kubectl rollout history deployment/web -n ch06-practice
```

이전 버전으로 롤백:

```bash
kubectl rollout undo deployment/web -n ch06-practice
```

중요:

```text
rollout status
= 배포 상태 확인

rollout history
= 배포 이력 확인

rollout undo
= 이전 버전으로 복구
```

명령어 전체를 외우기보다 역할과 사용 상황을 이해한다.

---

# 14. Service가 필요한 이유

Pod는 삭제되고 재생성될 수 있다.

예:

```text
Pod1 = 10.244.1.20
```

Pod가 삭제된 후:

```text
새 Pod = 10.244.2.30
```

처럼 IP가 바뀔 수 있다.

따라서 애플리케이션이 Pod IP에 직접 의존하면 안 된다.

그래서 Service를 사용한다.

```text
        Service
           ↓
   ┌───────┼───────┐
   ↓       ↓       ↓
 Pod1    Pod2    Pod3
```

Service는 Pod 앞에서 안정적인 접근점을 제공한다.

---

# 15. Service가 Pod를 찾는 방법

Service 역시 Label과 Selector를 사용한다.

Pod:

```text
label:
app=web
```

Service:

```text
selector:
app=web
```

따라서:

```text
Service
selector app=web
       ↓
app=web Label을 가진 Pod
```

로 연결된다.

---

# 16. ClusterIP Service 생성

파일:

```bash
vi web-service.yaml
```

```yaml
apiVersion: v1
kind: Service

metadata:
  name: web-service
  namespace: ch06-practice

spec:
  selector:
    app: web

  ports:
    - port: 80
      targetPort: 80
```

적용:

```bash
kubectl apply -f web-service.yaml
```

확인:

```bash
kubectl get svc -n ch06-practice
```

`type`을 지정하지 않으면 기본값은:

```text
ClusterIP
```

이다.

---

# 17. port와 targetPort

```yaml
ports:
  - port: 80
    targetPort: 80
```

의미:

```text
Service :80
     ↓
Pod :80
```

즉:

```text
port
= Service가 받는 포트

targetPort
= 실제 Pod Container가 받는 포트
```

이다.

---

# 18. EndpointSlice

Service가 실제로 어떤 Pod와 연결되는지 확인:

```bash
kubectl get endpointslice -n ch06-practice
```

구조:

```text
Service
   ↓
EndpointSlice
   ↓
Pod IP + Port
```

예:

```text
web-service
     ↓
EndpointSlice
     ↓
10.244.1.x:80
10.244.2.x:80
10.244.2.x:80
```

즉 EndpointSlice에는 Service 뒤에 실제로 연결 가능한 Endpoint 정보가 들어있다.

---

# 19. ClusterIP 통신 실습

임시 curl Pod 생성:

```bash
kubectl run curl-test \
  -n ch06-practice \
  --image=curlimages/curl \
  -it --rm -- sh
```

Pod 내부:

```bash
curl web-service
```

정상이라면 nginx 기본 HTML이 출력된다.

구조:

```text
curl-test Pod
      ↓
web-service
      ↓
nginx Pod
```

즉 ClusterIP는 Kubernetes Cluster 내부 통신에 사용할 수 있다.

---

# 20. WEB Pod → WAS Service 구조

실제 3-Tier 구조에서도 같은 원리를 사용한다.

```text
WEB Pod
   ↓
WAS Service
   ↓
WAS Pod
```

WEB Pod가 WAS Pod IP에 직접 연결하지 않는다.

```text
WAS Pod IP
= 재생성되면 바뀔 수 있음
```

따라서:

```text
http://was-service
```

처럼 Service 이름을 사용한다.

CoreDNS가 내부에서 Service 이름을 해석할 수 있도록 지원한다.

---

# 21. Service 종류

## ClusterIP

```text
Kubernetes Cluster 내부 통신
```

구조:

```text
Pod
 ↓
ClusterIP Service
 ↓
Pod
```

기본 Service 타입이다.

---

## NodePort

클러스터 외부에서:

```text
Node IP + NodePort
```

로 접근할 수 있다.

구조:

```text
외부
 ↓
Node IP : NodePort
 ↓
Service
 ↓
Pod
```

---

## LoadBalancer

Cloud 환경에서 외부 Load Balancer와 연동한다.

구조:

```text
인터넷
 ↓
Cloud Load Balancer
 ↓
Service
 ↓
Pod
```

---

# 22. Service 타입이 나뉜 이유

각 Service 타입은 접근 범위와 목적이 다르다.

```text
ClusterIP
= Cluster 내부에서 접근하고 싶다.

NodePort
= Cluster 밖에서도 Node IP와 Port로 접근하고 싶다.

LoadBalancer
= Cloud Load Balancer를 이용해 외부 서비스를 안정적으로 공개하고 싶다.
```

개념적으로:

```text
LoadBalancer
    ↓
NodePort
    ↓
ClusterIP
    ↓
Pod
```

형태로 이해할 수 있다.

---

# 23. NodePort Service 생성

파일:

```bash
vi web-nodeport.yaml
```

```yaml
apiVersion: v1
kind: Service

metadata:
  name: web-nodeport
  namespace: ch06-practice

spec:
  type: NodePort

  selector:
    app: web

  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

적용:

```bash
kubectl apply -f web-nodeport.yaml
```

확인:

```bash
kubectl get svc -n ch06-practice
```

예:

```text
web-service    ClusterIP
web-nodeport   NodePort    80:30080/TCP
```

---

# 24. NodePort 외부 접근

Worker1:

```text
192.168.111.138
```

Worker2:

```text
192.168.111.134
```

Chrome:

```text
http://192.168.111.138:30080
```

또는:

```text
http://192.168.111.134:30080
```

curl 테스트:

```bash
curl http://192.168.111.138:30080
```

핵심:

```text
NodePort
= Node IP + NodePort
```

Pod IP가 아니다.

---

# 25. NodePort와 Pod 위치

예를 들어 실제 nginx Pod가 Worker2에 있어도:

```text
Chrome
 ↓
Worker1 IP:30080
 ↓
NodePort Service
 ↓
Worker2의 nginx Pod
```

처럼 전달될 수 있다.

즉 NodePort의 Node IP와 실제 요청을 처리하는 Pod의 Node가 반드시 같을 필요는 없다.

---

# 26. Service와 Deployment는 별개이다

중요한 구조:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

와:

```text
Service
   ↓
Pod
```

는 서로 다른 관계이다.

```text
Deployment 계열
= Pod 관리

Service
= Pod 접근
```

Service가 Pod를 생성하는 것이 아니고,
Deployment가 Service를 생성하는 것도 아니다.

---

# 27. 6장 전체 구조

```text
                     외부 사용자
                          ↓
                 NodePort / LoadBalancer
                          ↓
                       Service
                          ↓
                ┌─────────┼─────────┐
                ↓         ↓         ↓
              Pod1      Pod2      Pod3
                ↑         ↑         ↑
                └──── ReplicaSet ───┘
                          ↑
                      Deployment
```

---

# 28. 관리 흐름과 트래픽 흐름

## 관리 흐름

```text
Deployment
 ↓
ReplicaSet
 ↓
Pod
 ↓
Container
```

## 트래픽 흐름

```text
Service
 ↓
Pod
```

이 둘을 구분하는 것이 중요하다.

---

# 29. 장애 대응 기본 흐름

Service 접속이 되지 않는다면:

```text
Service 존재?
 ↓
Selector 확인
 ↓
EndpointSlice 확인
 ↓
Pod Running?
 ↓
Container가 실제 Port에서 Listen?
 ↓
애플리케이션 로그 확인
```

관련 명령:

```bash
kubectl get svc -n ch06-practice
```

```bash
kubectl describe svc web-service -n ch06-practice
```

```bash
kubectl get endpointslice -n ch06-practice
```

```bash
kubectl get pods -n ch06-practice -o wide
```

```bash
kubectl logs <POD_NAME> -n ch06-practice
```

---

# 30. 6장 핵심 질문과 답

## Q1. Pod란?

Container를 실행하는 Kubernetes의 기본 단위이다.

---

## Q2. ReplicaSet 역할은?

원하는 Pod 개수를 유지한다.

---

## Q3. Deployment 역할은?

애플리케이션의 배포, Rolling Update, Rollback 등을 관리한다.

---

## Q4. Deployment와 ReplicaSet 관계는?

```text
Deployment
 ↓
ReplicaSet
 ↓
Pod
```

Deployment가 ReplicaSet을 이용해 Pod 배포를 관리한다.

---

## Q5. Pod가 하나 죽으면?

ReplicaSet이 원하는 Pod 개수를 유지하기 위해 새로운 Pod를 생성한다.

---

## Q6. Service가 필요한 이유는?

Pod는 재생성되면서 IP가 바뀔 수 있기 때문에 안정적인 접근점이 필요하다.

---

## Q7. Service는 Pod를 어떻게 찾는가?

```text
Service Selector
 ↓
Pod Label
```

을 이용한다.

---

## Q8. EndpointSlice란?

Service 뒤에 실제로 연결 가능한 Pod IP와 Port 정보를 관리한다.

---

## Q9. ClusterIP는?

Cluster 내부 접근용 Service이다.

---

## Q10. NodePort는?

```text
Node IP + NodePort
```

를 통해 외부에서 접근할 수 있는 Service이다.

---

## Q11. LoadBalancer는?

Cloud Load Balancer와 연동해 외부에 Service를 노출한다.

---

## Q12. Deployment가 Service를 자동으로 만드는가?

아니다.

둘은 독립적인 Kubernetes Resource이다.

---

## Q13. Service → Pod와 Deployment → Pod의 차이는?

```text
Deployment → ReplicaSet → Pod
= 관리 관계

Service → Pod
= 트래픽 전달 관계
```

이다.

---

## Q14. WEB Pod가 WAS Pod를 호출할 때 WAS Pod IP를 직접 사용해야 하는가?

아니다.

```text
WEB Pod
 ↓
was-service
 ↓
WAS Pod
```

구조를 사용하는 것이 좋다.

Pod IP는 변경될 수 있기 때문이다.

---

## Q15. Rolling Update란?

기존 Pod를 한 번에 모두 종료하지 않고 새 버전 Pod를 점진적으로 생성하면서 기존 Pod를 교체하는 방식이다.

---

# 31. 실습 명령어 모음

## Namespace

```bash
kubectl create namespace ch06-practice
```

## Deployment 적용

```bash
kubectl apply -f web-deployment.yaml
```

## Deployment / ReplicaSet / Pod 확인

```bash
kubectl get deploy,rs,pod -n ch06-practice -o wide
```

## Pod 삭제

```bash
kubectl delete pod <POD_NAME> -n ch06-practice
```

## Watch

```bash
kubectl get pods -n ch06-practice -w
```

## Scale

```bash
kubectl scale deployment web --replicas=5 -n ch06-practice
```

## 이미지 변경

```bash
kubectl set image deployment/web nginx=nginx:1.27 -n ch06-practice
```

## Rollout 상태

```bash
kubectl rollout status deployment/web -n ch06-practice
```

## Rollout 이력

```bash
kubectl rollout history deployment/web -n ch06-practice
```

## Rollback

```bash
kubectl rollout undo deployment/web -n ch06-practice
```

## Service 적용

```bash
kubectl apply -f web-service.yaml
```

## Service 확인

```bash
kubectl get svc -n ch06-practice
```

## EndpointSlice

```bash
kubectl get endpointslice -n ch06-practice
```

## NodePort

```bash
kubectl apply -f web-nodeport.yaml
```

## 외부 테스트

```bash
curl http://192.168.111.138:30080
```

---

# 32. 6장 최종 핵심

6장에서 가장 중요한 것은 다음 두 줄이다.

```text
Deployment → ReplicaSet → Pod
```

```text
Service → Pod
```

첫 번째는:

```text
애플리케이션 관리
```

두 번째는:

```text
트래픽 전달
```

을 담당한다.

그리고 Service를 사용하는 가장 중요한 이유는:

```text
Pod가 재생성되면 IP가 바뀔 수 있기 때문에
Service라는 안정적인 접근점을 사용한다.
```

는 것이다.

---

# 33. 6장 학습 완료 체크

- [x] Pod 역할 이해
- [x] ReplicaSet 역할 이해
- [x] Deployment 역할 이해
- [x] Deployment → ReplicaSet → Pod 구조 이해
- [x] Label / Selector 이해
- [x] Pod 삭제 후 자동 복구 확인
- [x] Desired State 확인
- [x] Scale 실습
- [x] Rolling Update 실습
- [x] Rollback 실습
- [x] Service가 필요한 이유 이해
- [x] ClusterIP 생성
- [x] EndpointSlice 확인
- [x] ClusterIP 내부 통신 확인
- [x] NodePort 생성
- [x] Node IP + NodePort 외부 접근 이해
- [x] ClusterIP / NodePort / LoadBalancer 차이 이해
- [x] Service → Pod와 Deployment → Pod 관계 구분


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/2ecad6d0-f27f-47ba-a953-29dffdabf6ac" />








</details>

<details>
<summary><b> 7장 쿠버네티스 리소스의 관리와 설정</b></summary>

## Namespace
1. 하나의 Kubernetes 클러스터 안에서 리소스를 논리적으로구분하는 폴더 같은 공간
2. Namespace는 클러스터를 물리적으로 쪼개는 게 아니라, 논리적으로 구역을 나누는 것이다.
3. Namespace는 서버를 나누는 게 아니다.
4. Namespace : 관리상의 구역
5. Node : 실제 Pod가 실행되는 서버

## 현재 Namespace 확인하기
1. kubectl get namespaces
2. kubectl get ns
3. default : Namespace를 지정하지 않고 만든 리소스가 기본적으로 들어가는 곳
4. kube-system : Kubernetes 시스템 구성요소들이 사용하는 곳
5. kubectl get pods -n default
6. kubectl get pods -n kube-system

## Namespace 만들기
1. kubectl create namespace portfolio
2. kubectl get ns
3. kubectl run nginx-test --image=nginx -n portfolio
4. kubectl get pods -n portfolio

## Namespace 전체에서 보기
1. kubectl get pods -A
2. kubectl get pods --all-namespaces

## Namespace는 하나의 Kubernetes 클러스터 안에서 리소스를 프로젝트, 팀, 환경별로 논리적으로 구분하기 위한 공간이다.

## ConfigMap, Secret: 설정값을 Pod에 전달
1. ConfigMap과 Secret은 애플리케이션 설정값을 Container 이미지 밖으로 빼서 관리하는 방법이다.
2. 일반 설정 : configMap
3. 민감한 설정 : Secret
4. 같은 애플리케이션 이미지를 그대로 두고 설정만 바꿀 수 있다.
5. Secret이라고 해서 무조건 강력하게 암호화되어 안전하다는 뜻은 아니다.

## ConfigMap
1. Pod가 사용할 일반 설정 값을 저장하는 Kubernetes 리소스이다.

## ConfigMap 만들기
1. kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=LOG_LEVEL=INFO -n portfolio
2. kubectl create configmap :  ConfigMap 만들기
3. app-config : ConfigMap 이름
4. --from-literal : 값을 직접 입력해서 저장
5. APP_ENV=production, LOG_LEVEL=INFO : 실제 설정값
6. -n portfolio : portfolio Namespace에 생성
7. kubectl get configmap -n portfolio
8. kubectl get cm -n portfolio
9. kubectl describe configmap app-config -n portfolio
10. kubectl get configmap app-config -n portfolio -o yaml
11. ConfigMap = 설정값을 적어놓은 메모장
12. Pod = 아직 그 메모장을 읽지 않은 상태

## ConfigMap은 비밀번호가 아닌 일반 설정 값을 Kubernetes에서 따로 관리하고 Pod에 전달하기 위한 리소스이다.

## ConfigMap을 실제 Pod에 넣어보는 단계
1. YAML 파일 만들기
2. nano configmap-pod.yaml : 테스트용 Pod 만들기

## ConfigMap은 일반 설정값을 애플리케이션 이미지와 분리해 저장하고, Pod가 필요할 때 그 값을 받아서 사용하도록 하는 Kubernetes 리소스이다.

## Secret
1. ConfigMap : 일반 설정값
2. Secret : 민감한 설정값
3. 실제 프로젝트에서는 ConfigMap생성, Secret생성, Deploymen 생성, Service 생성만한다.

## Secret 만들기
1. kubectl create secret generic db-secret --from-literal=DB_USERNAME=appuser --from-literal=DB_PASSWORD=1234 -n portfolio
2. kubectl create secret generic : 일반적인 시크릿 생성
3. db-secret : 시크릿 이름
4. DB_USERNAME=appuser : 디비 사용자명
5. DB_PASSWD = 1234 : 디비 비밀번호
6. kubectl get secret -n portfolio
7. kubectl describe secret db-secret -n portfolio
8. Base64 인코딩이 되어있어서 값을 바로 보여주진 않는다.

## Secret은 DB 비밀번호나 토큰 같은 민감한 설정값을 애플리케이션과 분리해서 관리하고, Deployment의 Pod Template에서 참조해서 Conatiner에 전달하는 Kubernetes 리소스이다.


## 확인문제
# 7장 쿠버네티스 리소스의 관리와 설정 - 확인문제 + 정답

## 1. Namespace란 무엇인가요?

**정답:**  
하나의 Kubernetes 클러스터 안에서 리소스를 논리적으로 구분하기 위한 공간입니다.

---

## 2. Namespace는 Node를 물리적으로 분리하는 기능인가요?

**정답:**  
아닙니다. Namespace는 논리적인 구분이며, 서로 다른 Namespace의 Pod가 같은 Node에서 실행될 수도 있습니다.

---

## 3. Namespace를 지정하지 않고 리소스를 만들면 기본적으로 어느 Namespace에 생성되나요?

**정답:**  
`default` Namespace

---

## 4. Kubernetes 시스템 구성요소들이 주로 사용하는 Namespace는 무엇인가요?

**정답:**  
`kube-system`

---

## 5. Namespace 목록을 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl get namespaces
```

또는

```bash
kubectl get ns
```

---

## 6. 특정 Namespace의 Pod를 조회하려면 어떻게 하나요?

**정답:**

```bash
kubectl get pods -n <Namespace이름>
```

예:

```bash
kubectl get pods -n portfolio
```

---

## 7. 모든 Namespace의 Pod를 한 번에 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl get pods -A
```

또는

```bash
kubectl get pods --all-namespaces
```

---

## 8. `portfolio`라는 Namespace를 만드는 명령어는 무엇인가요?

**정답:**

```bash
kubectl create namespace portfolio
```

---

## 9. 서로 다른 Namespace에서 같은 이름의 리소스를 만들 수 있나요?

**정답:**  
네. Namespace가 다르면 같은 이름의 리소스가 각각 존재할 수 있습니다.

---

## 10. ConfigMap이란 무엇인가요?

**정답:**  
애플리케이션이 사용할 일반 설정값을 이미지와 분리해서 관리하는 Kubernetes 리소스입니다.

---

## 11. ConfigMap에는 어떤 값을 저장할 수 있나요?

**정답:**  
DB 주소, DB 포트, 실행 환경, 로그 레벨 등 일반 설정값을 저장할 수 있습니다.

---

## 12. ConfigMap에 저장할 수 있는 일반 설정의 예를 들어보세요.

**정답:**

```text
APP_ENV=production
LOG_LEVEL=INFO
DB_HOST=mysql-service
DB_PORT=3306
```

---

## 13. `APP_ENV`나 `LOG_LEVEL`이라는 이름은 Kubernetes가 정해놓은 이름인가요?

**정답:**  
아닙니다. 애플리케이션과 운영자가 필요에 따라 정하는 설정 이름입니다.

---

## 14. 일반적으로 환경변수 이름을 `APP_ENV`처럼 대문자와 언더바(`_`)로 작성하는 이유는 무엇인가요?

**정답:**  
환경변수 이름을 표현할 때 흔히 사용하는 관례이기 때문입니다.

---

## 15. ConfigMap을 사용하는 가장 중요한 이유는 무엇인가요?

**정답:**  
애플리케이션 이미지와 환경별 설정값을 분리하기 위해서입니다.

---

## 16. ConfigMap을 사용하면 개발환경과 운영환경에서 같은 이미지를 사용할 수 있나요?

**정답:**  
네. 이미지는 그대로 두고 ConfigMap의 설정값만 다르게 사용할 수 있습니다.

---

## 17. `app-config`라는 ConfigMap을 생성하는 예시는 무엇인가요?

**정답:**

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=INFO \
  -n portfolio
```

---

## 18. ConfigMap 목록을 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl get configmap -n portfolio
```

또는

```bash
kubectl get cm -n portfolio
```

---

## 19. ConfigMap의 실제 내용을 YAML 형태로 확인하려면 어떻게 하나요?

**정답:**

```bash
kubectl get configmap app-config -n portfolio -o yaml
```

---

## 20. ConfigMap을 직접 수정하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl edit configmap app-config -n portfolio
```

---

## 21. ConfigMap을 만들기만 하면 Pod가 자동으로 그 설정을 사용하나요?

**정답:**  
아닙니다. Pod 또는 Deployment의 Pod Template에서 해당 ConfigMap을 참조해야 합니다.

---

## 22. Pod에 환경변수를 정의할 때 사용하는 YAML 항목은 무엇인가요?

**정답:**  
`env`

---

## 23. `valueFrom`의 의미는 무엇인가요?

**정답:**  
환경변수 값을 YAML에 직접 적지 않고 다른 Kubernetes 리소스에서 가져오겠다는 의미입니다.

---

## 24. ConfigMap에서 값을 가져올 때 사용하는 항목은 무엇인가요?

**정답:**  
`configMapKeyRef`

---

## 25. 다음 구조의 의미는 무엇인가요?

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

**정답:**  
`app-config`라는 ConfigMap에서 `APP_ENV` 값을 가져와 Container의 `APP_ENV` 환경변수로 사용하겠다는 의미입니다.

---

## 26. ConfigMap 값을 환경변수로 사용하고 있는 실행 중인 Pod가 있을 때 ConfigMap만 수정하면 Pod의 환경변수가 즉시 변경되나요?

**정답:**  
일반적으로 환경변수로 주입된 값은 실행 중인 Container에 자동 반영되지 않습니다. 새 값을 사용하려면 Pod를 재생성하거나 Deployment를 재시작해야 합니다.

예:

```bash
kubectl rollout restart deployment <Deployment이름> -n <Namespace>
```

---

## 27. ConfigMap과 Pod는 서로 다른 Namespace에 있어도 바로 참조할 수 있나요?

**정답:**  
일반적인 ConfigMap 참조에서는 같은 Namespace에 있어야 합니다.

---

## 28. ConfigMap이 존재하지 않거나 필요한 Key가 없으면 Pod에서 어떤 오류가 발생할 수 있나요?

**정답:**  
`CreateContainerConfigError`가 발생할 수 있습니다.

---

## 29. Pod가 `CreateContainerConfigError` 상태일 때 원인을 확인하는 대표적인 명령어는 무엇인가요?

**정답:**

```bash
kubectl describe pod <Pod이름> -n <Namespace>
```

예:

```bash
kubectl describe pod configmap-test -n portfolio
```

---

## 30. `kubectl describe`에서 특히 확인해야 하는 부분은 무엇인가요?

**정답:**  
출력 맨 아래의 `Events` 부분입니다.

---

## 31. 실제 프로젝트에서도 ConfigMap 테스트를 위해 Pod를 항상 직접 만들어야 하나요?

**정답:**  
아닙니다. 실제 프로젝트에서는 보통 Deployment의 Pod Template에서 ConfigMap을 참조합니다.

---

## 32. Deployment를 사용하는 경우 ConfigMap 연결 설정은 어디에 작성하나요?

**정답:**  
Deployment의 Pod Template 내부에 작성합니다.

구조:

```text
spec
└─ template
   └─ spec
      └─ containers
         └─ env
```

예:

```yaml
spec:
  template:
    spec:
      containers:
        - name: spring
          image: my-spring-app:v1

          env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_ENV
```

---

## 33. Deployment에서 `replicas: 3`이고 ConfigMap을 참조하면 몇 개의 Pod가 ConfigMap 설정을 사용할 수 있나요?

**정답:**  
Deployment가 생성한 3개의 Pod 모두 같은 ConfigMap 설정을 사용할 수 있습니다.

```text
ConfigMap
   ↓
Pod 1
Pod 2
Pod 3
```

---

# Secret

## 34. Secret이란 무엇인가요?

**정답:**  
비밀번호, 토큰, 인증정보 등의 민감한 설정값을 일반 설정과 분리해서 관리하는 Kubernetes 리소스입니다.

---

## 35. Secret에 저장하기 적합한 값의 예는 무엇인가요?

**정답:**

```text
DB_USERNAME
DB_PASSWORD
API_TOKEN
인증서 관련 정보
```

---

## 36. ConfigMap과 Secret의 가장 기본적인 차이는 무엇인가요?

**정답:**

```text
ConfigMap = 일반 설정
Secret    = 민감한 설정
```

---

## 37. ConfigMap과 Secret을 분리하는 주된 이유는 무엇인가요?

**정답:**  
민감정보를 일반 설정과 분리하여 보안, 권한 관리, 설정 관리를 더 체계적으로 하기 위해서입니다.

---

## 38. `db-secret`이라는 Secret을 만드는 예시는 무엇인가요?

**정답:**

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USERNAME=appuser \
  --from-literal=DB_PASSWORD=1234 \
  -n portfolio
```

---

## 39. Secret 목록을 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl get secret -n portfolio
```

---

## 40. Secret의 상세 정보를 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl describe secret db-secret -n portfolio
```

---

## 41. Secret을 YAML 형태로 확인하는 명령어는 무엇인가요?

**정답:**

```bash
kubectl get secret db-secret -n portfolio -o yaml
```

---

## 42. Secret의 `data` 값이 `MTIzNA==`처럼 보이는 이유는 무엇인가요?

**정답:**  
기본적으로 Base64 방식으로 인코딩되어 표현되기 때문입니다.

예:

```text
1234
↓ Base64 인코딩
MTIzNA==
```

---

## 43. Base64는 암호화인가요?

**정답:**  
아닙니다. Base64는 데이터를 표현하기 위한 인코딩 방식입니다.

---

## 44. Kubernetes Secret에 값을 넣기만 하면 완벽하게 안전해지나요?

**정답:**  
아닙니다. 실제 운영에서는 RBAC, etcd 암호화, 외부 Secret Manager 등 추가적인 보안 대책을 사용할 수 있습니다.

---

## 45. Pod에서 Secret 값을 가져올 때 사용하는 항목은 무엇인가요?

**정답:**  
`secretKeyRef`

---

## 46. 다음 YAML의 의미는 무엇인가요?

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

**정답:**  
`db-secret`이라는 Secret에서 `DB_PASSWORD` 값을 가져와 Container의 `DB_PASSWORD` 환경변수로 사용한다는 의미입니다.

---

## 47. ConfigMap과 Secret을 각각 Pod에 연결할 때 사용하는 참조 이름은 무엇인가요?

**정답:**

```text
ConfigMap → configMapKeyRef
Secret    → secretKeyRef
```

---

## 48. 실제 프로젝트에서 일반 설정과 민감 설정을 어떻게 나누는 것이 좋나요?

**정답:**

### ConfigMap

```text
APP_ENV
LOG_LEVEL
DB_HOST
DB_PORT
```

### Secret

```text
DB_USERNAME
DB_PASSWORD
API_TOKEN
```

---

# 실제 프로젝트 연결

## 49. 실제 프로젝트에서 우리가 직접 만드는 Kubernetes 리소스의 예는 무엇인가요?

**정답:**

```text
Namespace
ConfigMap
Secret
Deployment
Service
```

---

## 50. Deployment를 만들었을 때 보통 직접 만들지 않아도 자동으로 생성되는 것은 무엇인가요?

**정답:**

```text
Deployment
    ↓
ReplicaSet 자동 생성
    ↓
Pod 자동 생성
```

따라서 직접 만드는 것은 보통 Deployment이고, ReplicaSet과 Pod는 Deployment에 의해 관리됩니다.

---

## 51. 실제 프로젝트에서 ConfigMap과 Secret을 사용하는 전체 흐름은 어떻게 되나요?

**정답:**

```text
ConfigMap / Secret 생성
        ↓
Deployment 작성
        ↓
Deployment의 Pod Template에서
ConfigMap / Secret 참조
        ↓
ReplicaSet 자동 생성
        ↓
Pod 자동 생성
        ↓
Container가 설정값을 환경변수로 사용
```

---

## 52. ConfigMap과 Secret은 Deployment의 하위 리소스인가요?

**정답:**  
아닙니다.

ConfigMap, Secret, Deployment는 각각 독립적인 Kubernetes 리소스입니다.

Deployment의 Pod Template에서 ConfigMap과 Secret을 **참조해서 사용**합니다.

```text
ConfigMap ───┐
             ↓
Secret ────→ Pod
             ↑
Deployment → ReplicaSet → Pod
```

---

## 53. Namespace와 Deployment, ConfigMap, Secret의 관계를 간단히 설명하면?

**정답:**  
Namespace라는 논리적 공간 안에 Deployment, ConfigMap, Secret 등의 리소스를 각각 생성해서 관리할 수 있습니다.

```text
portfolio Namespace
│
├─ ConfigMap
├─ Secret
├─ Deployment
├─ Service
└─ Pod
```

---

## 54. 7장에서 배운 전체 구조를 표현하면 어떻게 되나요?

**정답:**

```text
Kubernetes Cluster
│
└─ Namespace
   │
   ├─ ConfigMap ───────┐
   │                   ↓
   ├─ Secret ────────→ Pod
   │                   ↑
   ├─ Deployment
   │     ↓
   │  ReplicaSet
   │     ↓
   │    Pod
   │     ↓
   │  Container
   │
   └─ Service
         ↓
       Pod에 접근
```

---

# 7장 핵심 압축

## Namespace

```text
Kubernetes 리소스를 논리적으로 구분하는 공간
```

예:

```text
default
kube-system
portfolio
dev
prod
```

---

## ConfigMap

```text
일반 설정값 관리
```

예:

```text
APP_ENV
LOG_LEVEL
DB_HOST
DB_PORT
```

Pod에서 가져올 때:

```yaml
configMapKeyRef
```

---

## Secret

```text
민감한 설정값 관리
```

예:

```text
DB_USERNAME
DB_PASSWORD
API_TOKEN
```

Pod에서 가져올 때:

```yaml
secretKeyRef
```

---

# 실제 프로젝트 기본 흐름

```text
Namespace 생성
        ↓
ConfigMap / Secret 생성
        ↓
Deployment 생성
        ↓
ReplicaSet 자동 생성
        ↓
Pod 자동 생성
        ↓
Pod가 ConfigMap / Secret 사용
        ↓
Service가 Pod에 접근
```

---

# 반드시 기억할 구조

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

그리고 옆에서 Pod를 돕는 리소스:

```text
ConfigMap
    ↓
일반 설정 전달

Secret
    ↓
민감 설정 전달

Service
    ↓
Pod에 접근

Namespace
    ↓
리소스들을 논리적으로 구분
```

---

# 한 줄 정리

> **Namespace는 리소스를 구분하고, ConfigMap과 Secret은 Pod가 사용할 설정을 외부에서 제공하며, Deployment는 ReplicaSet과 Pod를 생성·관리하고, Service는 해당 Pod에 접근할 수 있는 안정적인 창구를 제공합니다.**

# Kubernetes 7장 - Namespace, ConfigMap, Secret

## 1. 7장 전체 핵심 구조

7장의 핵심은 Kubernetes Resource를 논리적으로 분리하고,
애플리케이션 설정과 민감정보를 Pod와 분리해서 관리하는 것이다.

```text
                  Namespace
                     │
        ┌────────────┼────────────┐
        │            │            │
    ConfigMap     Deployment     Secret
        │            │            │
        │        ReplicaSet       │
        │            │            │
        └──────────→ Pod ←────────┘
```

핵심:

```text
Namespace
= 리소스의 논리적 구역

ConfigMap
= 일반 설정정보

Secret
= 민감정보
```

---

# 2. Namespace란?

Namespace는 하나의 Kubernetes Cluster 안에서
Resource를 논리적으로 구분하는 기능이다.

예:

```text
Kubernetes Cluster

├─ Namespace: dev
│  ├─ Deployment
│  ├─ Service
│  └─ Pod
│
├─ Namespace: staging
│  ├─ Deployment
│  ├─ Service
│  └─ Pod
│
└─ Namespace: prod
   ├─ Deployment
   ├─ Service
   └─ Pod
```

Namespace는 물리적으로 Node를 분리하는 기능이 아니다.

```text
Namespace
= 논리적 분리

Node
= 실제 Pod가 실행되는 서버
```

따라서:

```text
dev Pod  → worker1
prod Pod → worker1
```

처럼 서로 다른 Namespace의 Pod가 같은 Node에서 실행될 수도 있다.

---

# 3. Namespace가 필요한 이유

하나의 Cluster 안에 많은 Resource가 존재하면 관리가 복잡해진다.

예:

```text
web
api
db
redis
prometheus
grafana
```

Namespace를 사용하면:

```text
project-a
├─ web
├─ api
└─ db

project-b
├─ web
├─ api
└─ db
```

처럼 구분할 수 있다.

Namespace가 다르면 같은 Resource 이름도 사용할 수 있다.

```text
dev Namespace
└─ Deployment: web

prod Namespace
└─ Deployment: web
```

---

# 4. Namespace 확인

전체 Namespace 확인:

```bash
kubectl get namespaces
```

줄임:

```bash
kubectl get ns
```

기본적으로 존재하는 Namespace 예:

```text
default
kube-system
kube-public
kube-node-lease
```

---

# 5. default Namespace

명령어에서 Namespace를 지정하지 않으면 일반적으로
`default` Namespace를 사용한다.

```bash
kubectl get pods
```

는 사실상:

```bash
kubectl get pods -n default
```

를 조회하는 것과 같다.

특정 Namespace:

```bash
kubectl get pods -n ch07-practice
```

전체 Namespace:

```bash
kubectl get pods -A
```

자세히:

```bash
kubectl get pods -A -o wide
```

---

# 6. Namespace 생성 실습

```bash
kubectl create namespace ch07-practice
```

확인:

```bash
kubectl get ns
```

Pod 생성:

```bash
kubectl run namespace-test \
  --image=nginx \
  -n ch07-practice
```

확인:

```bash
kubectl get pods -n ch07-practice
```

Node까지 확인:

```bash
kubectl get pods -n ch07-practice -o wide
```

여기서:

```text
Namespace = Resource가 속한 논리적 공간
Node      = Pod가 실제 실행되는 서버
```

라는 차이를 이해한다.

---

# 7. Namespace와 Scheduling은 다르다

Namespace를:

```text
dev
prod
```

로 나눴다고 해서 자동으로:

```text
dev  → worker1
prod → worker2
```

처럼 Node가 분리되는 것은 아니다.

Node 선택은 이후 배우는:

```text
nodeSelector
Node Affinity
Taint / Toleration
```

같은 Scheduling 기능과 관련된다.

---

# 8. ConfigMap이란?

ConfigMap은 애플리케이션의 일반 설정값을
Container Image와 분리해서 관리하기 위한 Resource이다.

예:

```text
APP_ENV=production
LOG_LEVEL=INFO
```

애플리케이션 Image 내부에 설정을 직접 넣는 대신:

```text
Application Image
       +
   ConfigMap
```

으로 분리한다.

구조:

```text
ConfigMap
    ↓
   Pod
    ↓
Container
```

---

# 9. ConfigMap에 넣을 수 있는 값

예:

```text
APP_ENV
LOG_LEVEL
DB_HOST
API_URL
SPRING_PROFILES_ACTIVE
```

등이다.

중요:

```text
APP_ENV
LOG_LEVEL
```

은 Kubernetes 예약어가 아니다.

사용자가 원하는 이름으로 정의한 설정값이다.

---

# 10. ConfigMap 생성 실습

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=INFO \
  -n ch07-practice
```

확인:

```bash
kubectl get configmap -n ch07-practice
```

줄임:

```bash
kubectl get cm -n ch07-practice
```

내용 확인:

```bash
kubectl get configmap app-config \
  -n ch07-practice \
  -o yaml
```

예:

```yaml
data:
  APP_ENV: production
  LOG_LEVEL: INFO
```

---

# 11. ConfigMap 오타 실습

잘못 생성:

```bash
kubectl create configmap app-config \
  --from-literal=AAPP_ENV=production \
  --from-literal=LOG_LEVEL=INFO \
  -n ch07-practice
```

그러면 실제 key는:

```text
AAPP_ENV
```

가 된다.

Pod가:

```yaml
key: APP_ENV
```

를 요구하면 해당 key를 찾을 수 없다.

ConfigMap 삭제:

```bash
kubectl delete configmap app-config -n ch07-practice
```

정상 생성:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=INFO \
  -n ch07-practice
```

---

# 12. ConfigMap을 Pod 환경변수로 사용

예:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: configmap-test
  namespace: ch07-practice

spec:
  containers:
    - name: test
      image: busybox:1.36

      command:
        - sh
        - -c
        - |
          echo "APP_ENV=$APP_ENV"
          echo "LOG_LEVEL=$LOG_LEVEL"
          sleep 3600

      env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV

        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
```

적용:

```bash
kubectl apply -f configmap-ch07-pod.yaml
```

확인:

```bash
kubectl get pods -n ch07-practice
```

로그:

```bash
kubectl logs configmap-test -n ch07-practice
```

결과:

```text
APP_ENV=production
LOG_LEVEL=INFO
```

---

# 13. configMapKeyRef 구조

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

의미:

```text
Container 환경변수 이름
APP_ENV
    ↓
app-config ConfigMap에서
    ↓
APP_ENV key의 값을 가져온다.
```

즉:

```text
ConfigMap
APP_ENV=production
       ↓
Pod
       ↓
Container
APP_ENV=production
```

---

# 14. Container 내부에서 확인

```bash
kubectl exec -it configmap-test \
  -n ch07-practice \
  -- sh
```

확인:

```bash
echo $APP_ENV
```

```bash
echo $LOG_LEVEL
```

결과:

```text
production
INFO
```

---

# 15. 기존 Pod 수정 오류

이미 만들어진 Pod의 `command` 등을 YAML에서 변경하고:

```bash
kubectl apply -f configmap-ch07-pod.yaml
```

을 실행했을 때 다음 오류를 확인했다.

```text
pod updates may not change fields...
```

Pod의 많은 Spec 필드는 생성 이후 직접 변경할 수 없다.

따라서 직접 만든 Pod의 경우:

```text
Pod 삭제
↓
YAML 수정
↓
Pod 재생성
```

방식으로 처리한다.

```bash
kubectl delete pod configmap-test -n ch07-practice
```

```bash
kubectl apply -f configmap-ch07-pod.yaml
```

---

# 16. 직접 Pod와 Deployment 차이

직접 만든 Pod:

```text
Pod
```

Pod Spec을 크게 변경하기 어렵기 때문에
필요하면 삭제 후 재생성해야 한다.

Deployment 사용:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

Deployment의 Pod Template을 변경하면
새로운 Pod를 생성해 교체한다.

따라서 일반적인 애플리케이션 운영에서는
Pod를 직접 관리하기보다 Deployment를 사용한다.

---

# 17. ConfigMap 변경 실험

기존:

```text
LOG_LEVEL=INFO
```

ConfigMap 수정:

```bash
kubectl edit configmap app-config -n ch07-practice
```

변경:

```yaml
data:
  APP_ENV: production
  LOG_LEVEL: DEBUG
```

ConfigMap 확인:

```bash
kubectl get configmap app-config \
  -n ch07-practice \
  -o yaml
```

---

# 18. ConfigMap 변경과 환경변수

ConfigMap을 환경변수 방식으로 주입한 경우:

```text
ConfigMap 변경
      ↓
기존 Container 환경변수
자동 변경 X
```

Pod가 시작될 때:

```text
ConfigMap
↓
환경변수 생성
↓
Container 실행
```

되기 때문이다.

예:

```text
Pod 생성 시

LOG_LEVEL=INFO
       ↓
Container 환경변수
LOG_LEVEL=INFO
```

그 후 ConfigMap을:

```text
LOG_LEVEL=DEBUG
```

로 바꿔도 기존 Container는:

```text
LOG_LEVEL=INFO
```

를 계속 가지고 있을 수 있다.

새 값을 적용하려면 Pod를 재생성하거나
Deployment를 재시작하는 방식 등을 사용한다.

---

# 19. Secret이란?

Secret은 비밀번호, Token, API Key 등
민감한 정보를 관리하기 위한 Kubernetes Resource이다.

예:

```text
DB_USERNAME=appuser
DB_PASSWORD=1234
```

구조:

```text
Secret
   ↓
  Pod
   ↓
Container
```

---

# 20. ConfigMap과 Secret 차이

```text
ConfigMap
= 일반 설정정보

Secret
= 민감정보
```

예:

```text
ConfigMap
├─ APP_ENV
├─ LOG_LEVEL
└─ API_URL

Secret
├─ DB_PASSWORD
├─ API_KEY
└─ TOKEN
```

---

# 21. Secret 생성 실습

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USERNAME=appuser \
  --from-literal=DB_PASSWORD=1234 \
  -n ch07-practice
```

확인:

```bash
kubectl get secret -n ch07-practice
```

내용:

```bash
kubectl get secret db-secret \
  -n ch07-practice \
  -o yaml
```

예:

```yaml
data:
  DB_PASSWORD: MTIzNA==
  DB_USERNAME: YXBwdXNlcg==
```

---

# 22. Secret과 Base64

다음 값:

```text
MTIzNA==
```

은 암호화된 값이 아니다.

Base64 Encoding된 값이다.

확인:

```bash
echo MTIzNA== | base64 -d
```

결과:

```text
1234
```

따라서:

```text
Base64
≠ Encryption
```

이다.

Secret은 민감정보 전용 Kubernetes Resource이지만,
Base64만으로 정보가 보호되는 것은 아니다.

실무에서는 추가적으로:

```text
RBAC
etcd Encryption at Rest
외부 Secret Manager
```

등을 함께 고려할 수 있다.

---

# 23. Secret을 Pod 환경변수로 사용

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secret-test
  namespace: ch07-practice

spec:
  containers:
    - name: test
      image: busybox:1.36

      command:
        - sh
        - -c
        - |
          echo "DB_USERNAME=$DB_USERNAME"
          echo "DB_PASSWORD=$DB_PASSWORD"
          sleep 3600

      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_USERNAME

        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

적용:

```bash
kubectl apply -f secret-pod.yaml
```

확인:

```bash
kubectl get pods -n ch07-practice
```

로그:

```bash
kubectl logs secret-test -n ch07-practice
```

결과:

```text
DB_USERNAME=appuser
DB_PASSWORD=1234
```

---

# 24. secretKeyRef 구조

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

의미:

```text
Container 환경변수
DB_PASSWORD
      ↓
db-secret Secret
      ↓
DB_PASSWORD key
      ↓
Container에 값 전달
```

---

# 25. ConfigMap + Secret + Deployment 통합

7장에서 가장 중요한 구조:

```text
              ConfigMap
                  ↓
Deployment → ReplicaSet → Pod
                  ↑
                Secret
```

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: app
  namespace: ch07-practice

spec:
  replicas: 3

  selector:
    matchLabels:
      app: app

  template:
    metadata:
      labels:
        app: app

    spec:
      containers:
        - name: app
          image: busybox:1.36

          command:
            - sh
            - -c
            - |
              echo "APP_ENV=$APP_ENV"
              echo "LOG_LEVEL=$LOG_LEVEL"
              echo "DB_USERNAME=$DB_USERNAME"
              echo "DB_PASSWORD=$DB_PASSWORD"
              sleep 3600

          env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: APP_ENV

            - name: LOG_LEVEL
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: LOG_LEVEL

            - name: DB_USERNAME
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_USERNAME

            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: DB_PASSWORD
```

적용:

```bash
kubectl apply -f app-deployment.yaml
```

---

# 26. 통합 구조 확인

```bash
kubectl get deploy,rs,pod -n ch07-practice
```

실습 결과:

```text
Deployment
app

      ↓

ReplicaSet
app-57d84cc88c

      ↓

Pod 3개
```

Pod:

```text
app-57d84cc88c-b8vr5
app-57d84cc88c-tss5q
app-57d84cc88c-xcc72
```

---

# 27. 직접 생성과 자동 생성

이번 통합 실습 기준:

```text
직접 생성

Namespace
ConfigMap
Secret
Deployment
```

Deployment 생성 후:

```text
자동 생성

ReplicaSet
Pod
```

중요:

```text
ConfigMap은 Deployment가 만드는 것이 아니다.
Secret도 Deployment가 만드는 것이 아니다.
```

서로 독립적인 Kubernetes Resource이다.

---

# 28. Pod 로그 확인

```bash
kubectl logs <POD_NAME> -n ch07-practice
```

실습 결과:

```text
APP_ENV=production
LOG_LEVEL=DEBUG
DB_USERNAME=appuser
DB_PASSWORD=1234
```

따라서:

```text
ConfigMap
├─ APP_ENV
└─ LOG_LEVEL
       ↓

      Pod

       ↑
Secret
├─ DB_USERNAME
└─ DB_PASSWORD
```

연결이 정상적으로 이루어진 것을 확인했다.

---

# 29. 설정을 Deployment에 직접 넣지 않는 이유

이런 식으로 직접 넣을 수도 있다.

```yaml
env:
  - name: DB_PASSWORD
    value: "1234"
```

하지만 설정을 별도로 관리하려면:

```yaml
valueFrom:
  secretKeyRef:
```

또는:

```yaml
valueFrom:
  configMapKeyRef:
```

를 사용할 수 있다.

구조:

```text
Deployment
= 애플리케이션 실행 구조

ConfigMap
= 일반 설정

Secret
= 민감 설정
```

역할을 분리할 수 있다.

---

# 30. Namespace와 ConfigMap/Secret

Pod가 참조하는 ConfigMap과 Secret은
일반적으로 Pod와 같은 Namespace에 존재해야 한다.

예:

```text
Namespace: ch07-practice

├─ Pod
├─ ConfigMap: app-config
└─ Secret: db-secret
```

Namespace가 다르면 단순한 `configMapKeyRef` 또는
`secretKeyRef`로 직접 참조할 수 없다.

---

# 31. 일부러 ConfigMap 장애 만들기

정상:

```yaml
configMapKeyRef:
  name: app-config
  key: APP_ENV
```

일부러 오타:

```yaml
configMapKeyRef:
  name: app-config
  key: APP_EN_V
```

적용:

```bash
kubectl apply -f app-deployment.yaml
```

확인:

```bash
kubectl get deploy,rs,pod -n ch07-practice
```

새 Pod가 정상 실행되지 않는다.

---

# 32. 장애 원인 확인

Pod 확인:

```bash
kubectl get pods -n ch07-practice
```

문제 Pod 상세 확인:

```bash
kubectl describe pod <POD_NAME> -n ch07-practice
```

실제 실습에서 Events에 다음 오류가 발생했다.

```text
Error: couldn't find key APP_EN_V
in ConfigMap ch07-practice/app-config
```

뜻:

```text
Pod:
app-config에서 APP_EN_V 값을 주세요.

ConfigMap:
APP_EN_V라는 key가 존재하지 않습니다.
```

---

# 33. describe 명령어 형식

잘못 사용한 예:

```bash
kubectl describe pod pod/app-xxxxx -n ch07-practice
```

`pod`를 두 번 지정했다.

다음 둘 중 하나를 사용한다.

```bash
kubectl describe pod app-xxxxx -n ch07-practice
```

또는:

```bash
kubectl describe pod/app-xxxxx -n ch07-practice
```

기본 형식:

```text
kubectl describe <RESOURCE> <NAME>
```

또는:

```text
kubectl describe <RESOURCE>/<NAME>
```

---

# 34. Events 읽기

실습 결과:

```text
Normal   Scheduled
Successfully assigned ... to worker1
```

의미:

```text
Scheduler가 worker1에 Pod 배치 성공
```

다음:

```text
Normal Pulled
Container image "busybox:1.36" already present
```

의미:

```text
Container Image 문제 없음
```

마지막:

```text
Warning Failed
couldn't find key APP_EN_V in ConfigMap
```

의미:

```text
ConfigMap 참조 실패
```

따라서 문제 범위를 좁힐 수 있다.

```text
Scheduler 문제 X
Image 문제 X
ConfigMap 문제 O
```

---

# 35. CreateContainerConfigError 장애 대응

Pod 상태가:

```text
CreateContainerConfigError
```

라면 먼저:

```bash
kubectl describe pod <POD_NAME> -n ch07-practice
```

를 확인한다.

점검 대상:

```text
ConfigMap 이름
ConfigMap key

Secret 이름
Secret key
```

장애 대응 흐름:

```text
Pod Running 아님
       ↓
kubectl get pods
       ↓
CreateContainerConfigError
       ↓
kubectl describe pod
       ↓
Events 확인
       ↓
ConfigMap / Secret 확인
       ↓
YAML 수정
       ↓
kubectl apply
```

---

# 36. logs와 describe의 차이

Container가 정상 실행된 후
애플리케이션 내부 오류가 있다면:

```bash
kubectl logs
```

가 중요하다.

하지만 Container 자체가 시작하지 못했다면:

```bash
kubectl describe pod
```

의 `Events`가 매우 중요하다.

예:

```text
CreateContainerConfigError
ImagePullBackOff
FailedMount
Scheduling 문제
```

등은 `describe`를 먼저 확인할 수 있다.

---

# 37. 6장 Rolling Update와 연결

Deployment YAML의 Pod Template을 변경하면:

```text
Deployment
     ↓
새 ReplicaSet 생성
     ↓
새 Pod 생성
```

된다.

이번 장애에서는:

```text
기존 정상 Pod
     ↓
계속 서비스

새 ReplicaSet
     ↓
새 Pod 생성 시도
     ↓
ConfigMap key 오류
     ↓
새 Pod 실행 실패
```

가 발생했다.

즉 6장의 Deployment/Rolling Update와
7장의 ConfigMap이 연결된다.

---

# 38. 6장 + 7장 전체 구조

```text
                        Namespace
                           │
              ┌────────────┼─────────────┐
              │            │             │
          ConfigMap    Deployment      Secret
              │            │             │
              │        ReplicaSet        │
              │            │             │
              └──────────→ Pod ←─────────┘
                           ↑
                        Service
```

역할:

```text
Deployment
= Pod 배포 및 관리

ReplicaSet
= 원하는 Pod 수 유지

Pod
= Container 실행

Service
= Pod 접근

ConfigMap
= 일반 설정 제공

Secret
= 민감정보 제공

Namespace
= Resource 논리적 구분
```

---

# 39. 5~7장 연결

## 5장

Kubernetes Cluster 동작 구조

```text
kubectl
   ↓
API Server
   ↓
Controller / Scheduler
   ↓
kubelet
   ↓
Container Runtime
```

## 6장

애플리케이션 실행 및 접근

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
   ↑
Service
```

## 7장

설정 및 민감정보 제공

```text
ConfigMap → Pod ← Secret
```

결국:

```text
                 ConfigMap
                     ↓
kubectl → API Server → Deployment → ReplicaSet → Pod
                                              ↑
                                            Secret

Client → Service ───────────────────────────→ Pod
```

처럼 서로 연결된다.

---

# 40. 7장 핵심 질문과 답

## Q1. Namespace란?

Kubernetes Cluster 안의 Resource를 논리적으로 구분하는 공간이다.

---

## Q2. Namespace가 다르면 Node도 다른가?

아니다.

Namespace는 논리적인 분리이고,
Node는 실제 Pod가 실행되는 서버이다.

---

## Q3. ConfigMap이란?

애플리케이션의 일반 설정을
Pod/Image와 분리해서 관리하는 Resource이다.

---

## Q4. Secret이란?

비밀번호, Token, API Key 등
민감정보를 관리하기 위한 Resource이다.

---

## Q5. ConfigMap과 Secret의 가장 큰 차이는?

```text
ConfigMap = 일반 설정
Secret    = 민감정보
```

이다.

---

## Q6. Secret에 보이는 Base64 값은 암호화인가?

아니다.

Base64 Encoding일 뿐이다.

---

## Q7. ConfigMap을 환경변수로 사용했을 때 값을 바꾸면 기존 Pod의 환경변수도 바로 바뀌는가?

아니다.

이미 실행 중인 Container의 환경변수는 자동으로 변경되지 않는다.

새 값을 사용하려면 Pod 재생성/재시작 등이 필요하다.

---

## Q8. Deployment가 ConfigMap과 Secret을 자동 생성하는가?

아니다.

모두 독립적인 Kubernetes Resource이다.

---

## Q9. Deployment가 자동 생성하는 것은?

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

이다.

---

## Q10. ConfigMap key를 잘못 지정하면 어떤 문제가 발생할 수 있는가?

Pod가 Container 실행 설정을 만들지 못해:

```text
CreateContainerConfigError
```

상태가 발생할 수 있다.

---

## Q11. CreateContainerConfigError가 발생하면 무엇을 먼저 확인하는가?

```bash
kubectl describe pod <POD_NAME>
```

의 Events를 확인한다.

ConfigMap/Secret 이름과 key 오타 등을 점검한다.

---

## Q12. Pod가 Container 시작도 못 했다면 logs와 describe 중 무엇이 더 중요한가?

대체로:

```text
kubectl describe pod
```

의 Events를 먼저 확인한다.

---

## Q13. ConfigMap은 왜 사용하는가?

설정값을 Container Image와 분리해
환경별 설정을 관리하기 쉽게 하기 위해 사용한다.

---

## Q14. WEB/WAS 프로젝트에서는 어떻게 사용할 수 있는가?

예:

```text
ConfigMap
├─ SPRING_PROFILES_ACTIVE
├─ DB_HOST
└─ LOG_LEVEL

Secret
├─ DB_USERNAME
└─ DB_PASSWORD

           ↓

       WAS Pod
```

처럼 사용할 수 있다.

---

# 41. 주요 명령어 모음

Namespace:

```bash
kubectl get ns
```

```bash
kubectl create namespace ch07-practice
```

전체 Namespace Pod:

```bash
kubectl get pods -A
```

ConfigMap:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=LOG_LEVEL=INFO \
  -n ch07-practice
```

```bash
kubectl get cm -n ch07-practice
```

```bash
kubectl get configmap app-config \
  -n ch07-practice \
  -o yaml
```

```bash
kubectl edit configmap app-config -n ch07-practice
```

Secret:

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USERNAME=appuser \
  --from-literal=DB_PASSWORD=1234 \
  -n ch07-practice
```

```bash
kubectl get secret -n ch07-practice
```

```bash
kubectl get secret db-secret \
  -n ch07-practice \
  -o yaml
```

Base64 Decode:

```bash
echo MTIzNA== | base64 -d
```

Deployment:

```bash
kubectl apply -f app-deployment.yaml
```

```bash
kubectl get deploy,rs,pod -n ch07-practice
```

로그:

```bash
kubectl logs <POD_NAME> -n ch07-practice
```

Pod 내부:

```bash
kubectl exec -it <POD_NAME> \
  -n ch07-practice \
  -- sh
```

장애 확인:

```bash
kubectl describe pod <POD_NAME> -n ch07-practice
```

---

# 42. 7장 장애 대응 체크리스트

Pod가 정상 실행되지 않으면:

```text
1. kubectl get pods

2. Pod STATUS 확인

3. kubectl describe pod

4. Events 확인

5. ConfigMap 존재 확인

6. ConfigMap key 확인

7. Secret 존재 확인

8. Secret key 확인

9. Namespace 확인

10. YAML 수정 후 apply
```

---

# 43. 7장 최종 핵심

반드시 기억할 구조:

```text
Namespace
= Kubernetes Resource의 논리적 구역
```

```text
ConfigMap
= 일반 설정
```

```text
Secret
= 민감정보
```

그리고:

```text
              ConfigMap
                  ↓
Deployment → ReplicaSet → Pod
                  ↑
                Secret
```

여기에 6장의 Service까지 붙이면:

```text
              ConfigMap
                  ↓
Deployment → ReplicaSet → Pod ← Service
                  ↑
                Secret
```

정확히는:

```text
Deployment → ReplicaSet → Pod
= 관리 관계

Service → Pod
= 트래픽 관계

ConfigMap → Pod
= 일반 설정 제공

Secret → Pod
= 민감정보 제공
```

이다.

---

# 44. 7장 학습 완료 체크

- [x] Namespace 개념 이해
- [x] Namespace와 Node 차이 이해
- [x] Namespace 생성 실습
- [x] `-n`과 `-A` 이해
- [x] ConfigMap이 필요한 이유 이해
- [x] ConfigMap 생성 실습
- [x] ConfigMap 환경변수 주입
- [x] ConfigMap 변경 실험
- [x] 기존 Container 환경변수 자동 변경 여부 확인
- [x] Secret이 필요한 이유 이해
- [x] Secret 생성 실습
- [x] Base64가 암호화가 아님을 확인
- [x] Secret 환경변수 주입
- [x] ConfigMap + Secret + Deployment 통합
- [x] Deployment → ReplicaSet → Pod 구조 복습
- [x] ConfigMap key 오타 장애 생성
- [x] CreateContainerConfigError 확인
- [x] `kubectl describe pod` Events 분석
- [x] ConfigMap 장애 원인 추적
- [x] 6장과 7장 연결 이해

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/65bc8e93-3444-49b8-886a-a3171a3e0b60" />

</details>

<details>
<summary><b> 8장 인그레스(Ingress)</b></summary>

# Kubernetes 8장 - Ingress

## 1. 8장 전체 핵심 구조

Ingress는 여러 Service를 HTTP/HTTPS의 Host와 Path를 기준으로 라우팅하기 위한 Kubernetes Resource이다.

중요:

```text
Ingress
= 라우팅 규칙

Ingress Controller
= 그 규칙을 읽고 실제 HTTP/HTTPS 요청을 처리하는 프로그램
```

전체 구조:

```text
사용자
   ↓
Ingress Controller
   ↓
Ingress Rule
   ↓
Service
   ↓
EndpointSlice
   ↓
Pod
```

---

# 2. Ingress가 필요한 이유

NodePort만 사용하면 애플리케이션마다 다른 포트를 사용할 수 있다.

예:

```text
APP1 → NodePort 30080
APP2 → NodePort 30081
APP3 → NodePort 30082
```

접속:

```text
http://node-ip:30080
http://node-ip:30081
http://node-ip:30082
```

서비스가 많아질수록 포트 관리가 복잡해진다.

Ingress를 사용하면:

```text
http://ch08.local/app1
http://ch08.local/app2
```

처럼 하나의 진입점에서 Host/Path에 따라 여러 Service로 분기할 수 있다.

```text
ch08.local
   │
   ├─ /app1 → app1-service
   │
   └─ /app2 → app2-service
```

---

# 3. Ingress와 Ingress Controller

## Ingress

Ingress는 Kubernetes Resource이다.

예:

```text
/app1 → app1-service
/app2 → app2-service
```

라는 규칙을 정의한다.

Ingress 자체가 트래픽을 처리하는 프로그램은 아니다.

---

## Ingress Controller

Ingress Controller는 Ingress Resource를 감시하고 실제 요청을 처리한다.

우리 실습에서는:

```text
F5 NGINX Ingress Controller
```

를 사용했다.

구조:

```text
Ingress Resource
= 규칙

        ↓ 읽음

NGINX Ingress Controller
= 실제 처리
```

---

# 4. 설정 흐름과 트래픽 흐름

Ingress에서는 두 흐름을 구분해야 한다.

## 설정 흐름

```text
kubectl apply
      ↓
API Server
      ↓
Ingress Resource 저장
      ↓
Ingress Controller가 API Server 감시
      ↓
Ingress 규칙 발견
      ↓
NGINX 설정 반영
```

Ingress Controller가 로컬 YAML 파일을 직접 읽는 것이 아니다.

`kubectl`이 API Server에 Resource를 등록하고,
Controller가 API Server를 통해 Resource를 감시한다.

---

## 실제 트래픽 흐름

```text
사용자
 ↓
Ingress Controller
 ↓
Ingress 규칙에 따라 판단
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
```

즉:

```text
Control Plane 흐름
= 설정을 등록하고 감시하는 과정

Data Plane 흐름
= 실제 사용자 요청이 이동하는 과정
```

이다.

---

# 5. Ingress Controller 설치와 애플리케이션 배포 구분

이번 실습에서는 공부를 위해 Ingress Controller 설치도 직접 했다.

## 클러스터 구축 단계

```text
NGINX Ingress Controller Deployment
        ↓
Controller Pod

Controller용 NodePort Service
        ↓
외부 HTTP/HTTPS 진입점
```

설치 과정에서 사용:

```bash
kubectl apply -f deployments/common/ns-and-sa.yaml
kubectl apply -f deployments/rbac/rbac.yaml
kubectl apply -f deployments/common/nginx-config.yaml
kubectl apply -f deployments/common/ingress-class.yaml
kubectl apply -f deployments/deployment/nginx-ingress.yaml
kubectl create -f deployments/service/nodeport.yaml
```

---

## 애플리케이션 배포 단계

```text
Namespace
↓
Deployment
↓
Service
↓
Ingress
```

실무에서 기존 Kubernetes Cluster를 사용하는 경우
Ingress Controller는 이미 설치되어 있을 수 있다.

그러면 애플리케이션 담당자는 주로:

```text
Deployment
Service
Ingress
```

를 구성한다.

---

# 6. 8장 Namespace

실습용 Namespace:

```bash
kubectl create namespace ch08-practice
```

YAML로 표현하면:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: ch08-practice
```

Namespace는 Resource를 논리적으로 구분한다.

```text
Kubernetes Cluster

└─ ch08-practice
    ├─ Deployment app1
    ├─ Deployment app2
    ├─ Service app1-service
    ├─ Service app2-service
    └─ Ingress practice-ingress
```

Namespace는 Node를 물리적으로 분리하는 기능이 아니다.

---

# 7. APP1 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: app1
  namespace: ch08-practice

spec:
  replicas: 2

  selector:
    matchLabels:
      app: app1

  template:
    metadata:
      labels:
        app: app1

    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo:1.0

          args:
            - "-text=Hello from APP1"

          ports:
            - containerPort: 5678
```

의미:

```text
ch08-practice Namespace에
app1 Deployment 생성

Pod 2개 유지

Pod Label
app=app1

Container Image
hashicorp/http-echo:1.0

HTTP 요청 시
Hello from APP1 응답

Application Port
5678
```

구조:

```text
Deployment app1
       ↓
ReplicaSet
       ↓
 ┌─────┴─────┐
Pod          Pod
```

---

# 8. APP1 Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: app1-service
  namespace: ch08-practice

spec:
  selector:
    app: app1

  ports:
    - port: 80
      targetPort: 5678
```

뜻:

```text
app=app1 Label을 가진 Pod를 찾아라.

Service의 80번 포트로 받은 요청을
Pod의 5678번 포트로 전달해라.
```

구조:

```text
app1-service :80
       ↓
targetPort :5678
       ↓
app1 Pod :5678
```

---

# 9. Service type

Service YAML에:

```yaml
type:
```

을 작성하지 않았으므로 기본 타입인:

```text
ClusterIP
```

가 사용된다.

실습 결과:

```text
app1-service → ClusterIP
app2-service → ClusterIP
```

Ingress가 외부 노출을 담당하기 때문에
Backend Service는 ClusterIP로 사용할 수 있다.

```text
외부
 ↓
Ingress Controller
 ↓
ClusterIP Service
 ↓
Pod
```

---

# 10. Service의 핵심 포트

```yaml
ports:
  - port: 80
    targetPort: 5678
```

## port

```text
Service가 받는 포트
```

현재:

```text
Service :80
```

## targetPort

```text
실제 Pod Application이 요청을 받는 포트
```

현재:

```text
Pod :5678
```

따라서:

```text
Service :80
   ↓
Pod :5678
```

이다.

---

# 11. EndpointSlice

Service가 실제 어떤 Pod와 연결됐는지 확인:

```bash
kubectl get endpointslice -n ch08-practice
```

실습 결과:

```text
app1-service
→ 10.244.2.55:5678
→ 10.244.1.69:5678

app2-service
→ 10.244.1.70:5678
→ 10.244.2.56:5678
```

구조:

```text
Service
   ↓
EndpointSlice
   ↓
Pod IP + Port
```

---

# 12. Deployment selector와 Service selector

Deployment:

```yaml
selector:
  matchLabels:
    app: app1
```

뜻:

```text
어떤 Pod를 Deployment가 관리할 것인가?
```

Service:

```yaml
selector:
  app: app1
```

뜻:

```text
어떤 Pod로 트래픽을 보낼 것인가?
```

따라서:

```text
Deployment
selector app=app1
      ↓ 관리

     Pod
label app=app1

      ↑ 트래픽
Service
selector app=app1
```

이다.

---

# 13. Ingress 생성

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: practice-ingress
  namespace: ch08-practice

spec:
  ingressClassName: nginx

  rules:
    - host: ch08.local

      http:
        paths:
          - path: /app1
            pathType: Prefix

            backend:
              service:
                name: app1-service
                port:
                  number: 80

          - path: /app2
            pathType: Prefix

            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

---

# 14. Ingress apiVersion

```yaml
apiVersion: networking.k8s.io/v1
```

Ingress는 Kubernetes Networking API Group을 사용한다.

지금까지 비교:

```text
Namespace
→ v1

Service
→ v1

Deployment
→ apps/v1

Ingress
→ networking.k8s.io/v1
```

---

# 15. ingressClassName

```yaml
ingressClassName: nginx
```

뜻:

```text
이 Ingress는
nginx IngressClass에 연결된
Ingress Controller가 처리한다.
```

구조:

```text
Ingress
   ↓
ingressClassName: nginx
   ↓
IngressClass nginx
   ↓
NGINX Ingress Controller
```

---

# 16. Host

```yaml
host: ch08.local
```

HTTP 요청의 Host를 기준으로 라우팅한다.

테스트할 때:

```bash
curl -H "Host: ch08.local" ...
```

을 사용한 이유이다.

실제 DNS 환경에서:

```text
ch08.local
↓
Ingress Controller IP
```

가 정상 설정되어 있다면 브라우저가 Host Header를 자동으로 보낸다.

---

# 17. Path

```yaml
path: /app1
pathType: Prefix
```

`Prefix`이므로 `/app1` 경로를 기준으로 매칭한다.

예:

```text
/app1
/app1/
/app1/test
/app1/users
```

등이 해당 경로 규칙에 매칭될 수 있다.

---

# 18. Backend

```yaml
backend:
  service:
    name: app1-service
    port:
      number: 80
```

뜻:

```text
/app1 요청
↓
app1-service의 80번 포트
```

이다.

Ingress가 Pod IP를 직접 지정하지 않는다.

```text
Ingress
↓
Service
↓
Pod
```

구조를 사용한다.

---

# 19. APP1 전체 트래픽 흐름

```text
ch08.local/app1
       ↓
NGINX Ingress Controller
       ↓
Ingress 규칙
       ↓
app1-service :80
       ↓
EndpointSlice
       ↓
app1 Pod :5678
       ↓
Hello from APP1
```

---

# 20. APP2 전체 트래픽 흐름

```text
ch08.local/app2
       ↓
NGINX Ingress Controller
       ↓
Ingress 규칙
       ↓
app2-service :80
       ↓
EndpointSlice
       ↓
app2 Pod :5678
       ↓
Hello from APP2
```

---

# 21. Ingress Host 충돌 장애

처음에는:

```yaml
host: practice.local
```

을 사용했다.

하지만 기존 Ingress가 이미 같은 Host를 사용하고 있어서:

```text
Warning Rejected
All hosts are taken by other resources
```

오류가 발생했다.

확인:

```bash
kubectl describe ingress practice-ingress -n ch08-practice
```

Host를:

```yaml
host: ch08.local
```

로 변경했다.

다시 적용:

```bash
kubectl apply -f ch08-ingress.yaml
```

정상 Event:

```text
Normal AddedOrUpdated
Configuration for ch08-practice/practice-ingress was added or updated
```

---

# 22. Ingress Controller용 NodePort Service

Ingress Controller를 설치할 때 다음 파일을 적용했다.

```bash
kubectl create -f deployments/service/nodeport.yaml
```

이 파일은 우리가 직접 작성한 애플리케이션 Service YAML이 아니라
NGINX Ingress Controller에서 제공한 설치용 YAML이다.

이 Service의 역할:

```text
외부 사용자
   ↓
Controller용 NodePort Service
   ↓
Ingress Controller Pod
```

이다.

---

# 23. Controller Service 확인

```bash
kubectl get svc nginx-ingress -n nginx-ingress
```

실습 결과:

```text
80:31779/TCP
443:31075/TCP
```

해석:

```text
HTTP

Service Port 80
↓
NodePort 31779
```

```text
HTTPS

Service Port 443
↓
NodePort 31075
```

31779와 31075는 Service YAML에 nodePort를 직접 지정하지 않았기 때문에
Kubernetes가 자동으로 할당했다.

---

# 24. Namespace 이름과 Service 이름 구분

명령:

```bash
kubectl get svc nginx-ingress -n nginx-ingress
```

해석:

```text
svc nginx-ingress
= Service 이름

-n nginx-ingress
= Namespace 이름
```

현재 구조:

```text
Namespace: nginx-ingress
        ↓
Service: nginx-ingress
```

이므로 이름이 같아서 처음에는 헷갈릴 수 있다.

---

# 25. HTTP 테스트

Controller의 HTTP NodePort:

```text
31779
```

테스트:

```bash
curl -H "Host: ch08.local" \
  http://192.168.111.138:31779/app1
```

결과:

```text
Hello from APP1
```

APP2:

```bash
curl -H "Host: ch08.local" \
  http://192.168.111.138:31779/app2
```

결과:

```text
Hello from APP2
```

---

# 26. HTTP 전체 경로

```text
curl
 ↓
Worker IP :31779
 ↓
nginx-ingress NodePort Service :80
 ↓
NGINX Ingress Controller
 ↓
Ingress Rule
 ↓
app1-service :80
 ↓
app1 Pod :5678
```

중요:

```text
31779
= app1-service NodePort가 아니다.

Ingress Controller 앞의
NodePort Service 포트이다.
```

---

# 27. Annotation

Ingress의 기본 spec은:

```text
어디로 요청을 보낼 것인가?
```

를 정의한다.

Annotation은:

```text
Ingress Controller가 요청을
어떻게 처리할 것인가?
```

를 추가로 정의한다.

한 줄 정리:

```text
spec
= 어디로?

annotation
= 어떻게?
```

---

# 28. Annotation 실습

F5 NGINX Ingress Controller를 사용하기 때문에:

```text
nginx.org/...
```

형식을 사용한다.

```yaml
metadata:
  name: practice-ingress
  namespace: ch08-practice

  annotations:
    nginx.org/proxy-connect-timeout: "10s"
    nginx.org/proxy-read-timeout: "30s"
```

적용:

```bash
kubectl apply -f ch08-ingress.yaml
```

확인:

```bash
kubectl describe ingress practice-ingress -n ch08-practice
```

---

# 29. proxy-connect-timeout

```yaml
nginx.org/proxy-connect-timeout: "10s"
```

Backend Service/Pod와 연결을 맺기 위해 기다릴 시간을 설정한다.

---

# 30. proxy-read-timeout

```yaml
nginx.org/proxy-read-timeout: "30s"
```

Backend의 응답 데이터를 기다리는 시간을 설정한다.

---

# 31. SSL/TLS

기존 HTTP:

```text
http://ch08.local
```

HTTPS:

```text
https://ch08.local
```

HTTPS를 사용하려면:

```text
인증서
+
개인키
```

가 필요하다.

구조:

```text
Certificate
+
Private Key
    ↓
TLS Secret
    ↓
Ingress
    ↓
Ingress Controller
```

---

# 32. Self-Signed 인증서 생성

실습용 인증서:

```bash
openssl req \
  -x509 \
  -nodes \
  -days 365 \
  -newkey rsa:2048 \
  -keyout ch08.key \
  -out ch08.crt \
  -subj "/CN=ch08.local" \
  -addext "subjectAltName=DNS:ch08.local"
```

생성 확인:

```bash
ls -l ch08.key ch08.crt
```

---

# 33. TLS Secret

```bash
kubectl create secret tls ch08-tls \
  --cert=ch08.crt \
  --key=ch08.key \
  -n ch08-practice
```

확인:

```bash
kubectl get secret ch08-tls -n ch08-practice
```

실습 결과:

```text
NAME       TYPE                DATA
ch08-tls   kubernetes.io/tls   2
```

`DATA 2`는 일반적으로:

```text
tls.crt
tls.key
```

두 항목이 있다는 의미이다.

---

# 34. Ingress와 TLS Secret 연결

Ingress에 추가:

```yaml
spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - ch08.local

      secretName: ch08-tls
```

의미:

```text
ch08.local HTTPS 요청에
ch08-tls Secret의 인증서와 개인키를 사용한다.
```

---

# 35. TLS 전체 Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: practice-ingress
  namespace: ch08-practice

  annotations:
    nginx.org/proxy-connect-timeout: "10s"
    nginx.org/proxy-read-timeout: "30s"

spec:
  ingressClassName: nginx

  tls:
    - hosts:
        - ch08.local
      secretName: ch08-tls

  rules:
    - host: ch08.local

      http:
        paths:

          - path: /app1
            pathType: Prefix

            backend:
              service:
                name: app1-service
                port:
                  number: 80

          - path: /app2
            pathType: Prefix

            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

---

# 36. HTTPS 테스트

Controller HTTPS NodePort:

```text
31075
```

APP1:

```bash
curl -k \
  --resolve ch08.local:31075:192.168.111.138 \
  https://ch08.local:31075/app1
```

결과:

```text
Hello from APP1
```

APP2:

```bash
curl -k \
  --resolve ch08.local:31075:192.168.111.138 \
  https://ch08.local:31075/app2
```

결과:

```text
Hello from APP2
```

---

# 37. `-k`

Self-Signed 인증서는 공인 CA가 서명한 인증서가 아니므로
curl이 신뢰하지 않는다.

실습에서는:

```bash
curl -k
```

를 사용해 인증서 검증을 생략했다.

실제 서비스에서는 무조건 인증서 검증을 끄는 방식으로 운영하는 것이 아니다.

---

# 38. `--resolve`

```bash
--resolve ch08.local:31075:192.168.111.138
```

뜻:

```text
이번 curl 요청에서

ch08.local
     ↓
192.168.111.138

로 연결해라.
```

DNS를 직접 구성하지 않은 실습환경에서
DNS 역할을 임시로 대신한다.

---

# 39. HTTPS 전체 트래픽

```text
Client
 ↓ HTTPS
Worker IP :31075
 ↓
nginx-ingress NodePort Service :443
 ↓
NGINX Ingress Controller
 ↓
ch08-tls 인증서 사용
 ↓
Host / Path 확인
 ↓
app1-service 또는 app2-service
 ↓
Pod
```

---

# 40. IngressClass

하나의 Cluster에 여러 Ingress Controller가 존재할 수 있다.

예:

```text
Public NGINX Controller

Private NGINX Controller

다른 Controller
```

그러면 Ingress가 어느 Controller의 관리 대상인지 구분할 필요가 있다.

그 역할을 하는 것이:

```text
IngressClass
```

이다.

---

# 41. ingressClassName

Ingress:

```yaml
spec:
  ingressClassName: nginx
```

구조:

```text
practice-ingress
       ↓
ingressClassName: nginx
       ↓
IngressClass nginx
       ↓
NGINX Ingress Controller
```

---

# 42. IngressClass 확인

```bash
kubectl get ingressclass
```

자세히:

```bash
kubectl describe ingressclass nginx
```

또는:

```bash
kubectl get ingressclass nginx -o yaml
```

우리 환경의 Controller 식별자는:

```text
nginx.org/ingress-controller
```

형태이다.

---

# 43. IngressClass는 Controller가 아니다

중요:

```text
IngressClass
≠ Ingress Controller
```

IngressClass가 Controller를 실행하는 것이 아니다.

```text
IngressClass
= 어떤 Controller가 처리할 것인지 구분

Ingress Controller
= 실제 요청 처리
```

이다.

---

# 44. 여러 Controller 사용 예

예:

```text
Internet Service
      ↓
Ingress
ingressClassName: public
      ↓
Public Controller
```

사내용:

```text
Internal Service
      ↓
Ingress
ingressClassName: internal
      ↓
Internal Controller
```

처럼 구성할 수 있다.

---

# 45. Controller 하나가 여러 Ingress를 처리할 수 있다

애플리케이션마다 Controller를 설치할 필요는 없다.

```text
NGINX Ingress Controller
        ↑
        ├─ 쇼핑몰 Ingress
        ├─ 회원 Ingress
        ├─ 주문 Ingress
        └─ 관리자 Ingress
```

처럼 하나의 Controller가 여러 Ingress Resource를 처리할 수 있다.

---

# 46. 5~8장 전체 연결

## 5장

```text
kubectl
 ↓
API Server
 ↓
Controller / Scheduler
 ↓
kubelet
 ↓
Container Runtime
```

## 6장

```text
Deployment
 ↓
ReplicaSet
 ↓
Pod

Service
 ↓
Pod
```

## 7장

```text
ConfigMap → Pod ← Secret
```

## 8장

```text
Ingress Controller
 ↓
Service
 ↓
Pod
```

전체:

```text
                      ConfigMap
                          ↓
Deployment → ReplicaSet → Pod ← Secret
                          ↑
                       Service
                          ↑
                    Ingress Rule
                          ↑
                Ingress Controller
                          ↑
                        Client
```

---

# 47. 리소스 생성 순서

학습 및 일반적인 애플리케이션 구성 흐름:

```text
Namespace
    ↓
ConfigMap / Secret
    ↓
Deployment
    ↓
ReplicaSet 자동 생성
    ↓
Pod 자동 생성
    ↓
Service
    ↓
Ingress
```

Ingress Controller는 애플리케이션마다 만드는 것이 아니라
클러스터에 미리 준비되어 있을 수 있다.

---

# 48. 각 Resource 역할

```text
Namespace
= 리소스를 어디에 묶을 것인가

ConfigMap
= 일반 설정

Secret
= 민감 설정

Deployment
= Pod를 어떻게 운영할 것인가

ReplicaSet
= 원하는 Pod 수 유지

Pod
= Container 실행

Service
= Pod에 안정적으로 접근

Ingress
= HTTP/HTTPS 라우팅 규칙

IngressClass
= 어떤 Ingress Controller를 사용할지 구분

Ingress Controller
= 실제 HTTP/HTTPS 요청 처리
```

---

# 49. 8장 핵심 면접 질문

## Q1. Ingress가 필요한 이유는?

여러 Service를 Host와 Path를 기준으로 하나의 외부 진입점에서 라우팅하기 위해 사용한다.

---

## Q2. Ingress와 Ingress Controller 차이는?

```text
Ingress
= 라우팅 규칙

Ingress Controller
= 규칙을 읽어서 실제 트래픽을 처리
```

---

## Q3. Ingress가 Pod로 직접 트래픽을 보내는가?

일반적인 구성에서는 Service를 Backend로 사용한다.

```text
Ingress Controller
↓
Service
↓
Pod
```

---

## Q4. 왜 app1-service는 ClusterIP인가?

외부 요청은 Ingress Controller가 받고,
app1-service는 Cluster 내부에서 Pod로 전달하면 되기 때문이다.

---

## Q5. IngressClass는 무엇인가?

여러 Ingress Controller가 있을 때
어떤 Controller가 해당 Ingress를 처리할지 선택하는 기준이다.

---

## Q6. Annotation은 무엇인가?

Ingress Controller에게 세부 동작을 설정하기 위한 추가 설정이다.

예:

```text
timeout
redirect
기타 Controller별 설정
```

---

## Q7. TLS 인증서는 어디에 저장했는가?

Kubernetes TLS Secret에 저장했다.

```text
Certificate + Private Key
↓
TLS Secret
↓
Ingress
```

---

## Q8. 31075는 무엇인가?

Ingress Controller 앞의 NodePort Service에 Kubernetes가 자동 배정한 HTTPS NodePort이다.

```text
Node :31075
↓
Service :443
↓
Ingress Controller
```

---

## Q9. 31075를 우리가 직접 지정했는가?

아니다.

Controller용 Service YAML에서 `nodePort`를 지정하지 않았기 때문에
Kubernetes가 자동으로 할당했다.

---

## Q10. 실제 실무에서도 애플리케이션마다 Ingress Controller를 설치하는가?

보통 아니다.

클러스터 수준에서 Controller를 운영하고
여러 애플리케이션 Ingress가 이를 공유할 수 있다.

---

# 50. 주요 명령어

Namespace:

```bash
kubectl get ns
```

Deployment / Service:

```bash
kubectl get deploy,rs,pod,svc \
  -n ch08-practice \
  -o wide
```

EndpointSlice:

```bash
kubectl get endpointslice -n ch08-practice
```

Ingress:

```bash
kubectl get ingress -n ch08-practice
```

```bash
kubectl describe ingress practice-ingress \
  -n ch08-practice
```

IngressClass:

```bash
kubectl get ingressclass
```

Controller Service:

```bash
kubectl get svc nginx-ingress \
  -n nginx-ingress
```

HTTP:

```bash
curl -H "Host: ch08.local" \
  http://192.168.111.138:31779/app1
```

HTTPS:

```bash
curl -k \
  --resolve ch08.local:31075:192.168.111.138 \
  https://ch08.local:31075/app1
```

---

# 51. 8장 최종 핵심

가장 중요한 구조:

```text
사용자
   ↓
Ingress Controller
   ↓
Ingress 규칙
   ↓
Service
   ↓
EndpointSlice
   ↓
Pod
```

반드시 구분:

```text
Ingress
= 규칙

Ingress Controller
= 실제 프로그램
```

그리고:

```text
IngressClass
= 어떤 Controller가 처리할지 선택
```

HTTPS:

```text
Certificate + Private Key
        ↓
TLS Secret
        ↓
Ingress
        ↓
Ingress Controller
```

---

# 52. 8장 학습 완료 체크

- [x] Ingress가 필요한 이유 이해
- [x] NodePort와 Ingress 차이 이해
- [x] Ingress와 Ingress Controller 차이 이해
- [x] 설정 흐름과 실제 트래픽 흐름 구분
- [x] Namespace 생성
- [x] APP1 / APP2 Deployment 생성
- [x] ReplicaSet / Pod 자동 생성 확인
- [x] Service 생성
- [x] ClusterIP 이해
- [x] port / targetPort 이해
- [x] EndpointSlice 확인
- [x] Ingress 생성
- [x] Host 기반 라우팅 이해
- [x] Path 기반 라우팅 이해
- [x] Prefix 이해
- [x] Host 충돌 장애 확인
- [x] kubectl describe ingress 확인
- [x] HTTP `/app1` 테스트 성공
- [x] HTTP `/app2` 테스트 성공
- [x] Annotation 적용
- [x] TLS 인증서 생성
- [x] TLS Secret 생성
- [x] Ingress TLS 연결
- [x] HTTPS `/app1` 테스트 성공
- [x] HTTPS `/app2` 테스트 성공
- [x] Controller용 NodePort Service 이해
- [x] 31779 / 31075가 어디서 생겼는지 이해
- [x] IngressClass 이해
- [x] 여러 Ingress Controller 개념 이해
- [x] 5~8장 구조 연결


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/a9f3fea0-ba09-465f-9a8f-637101299b31" />


</details>





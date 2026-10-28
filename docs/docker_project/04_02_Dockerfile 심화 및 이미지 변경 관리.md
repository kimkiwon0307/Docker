
# 4장 2교시 — Dockerfile 심화 및 이미지 변경 관리

## 1. RUN, CMD, ENTRYPOINT 차이

Dockerfile에서 명령어를 실행하거나 컨테이너 실행 방법을 지정할 때 사용한다.

| 명령어 | 역할 | 실행 시점 |
|---|---|---|
| RUN | 빌드 과정에서 명령 실행 | 이미지 빌드 시 |
| CMD | 컨테이너 기본 실행 명령·인자 지정 | 컨테이너 시작 시 |
| ENTRYPOINT | 컨테이너 실행 진입점 지정 | 컨테이너 시작 시 |

### RUN

```dockerfile
RUN test -s index.html
```

- 이미지 빌드 중 명령어를 실행한다.
- 파일 생성, 패키지 설치, 파일 검사 등에 사용한다.
- 위 예시는 index.html이 존재하고 비어 있지 않은지 확인한다.

### CMD

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

- 컨테이너 시작 시 사용할 기본 명령어를 지정한다.
- docker run에서 다른 명령을 전달하면 기본 CMD를 대체할 수 있다.

### ENTRYPOINT

```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

- ENTRYPOINT는 실행할 프로그램을 지정한다.
- CMD는 해당 프로그램에 전달할 기본 인자를 지정할 수 있다.
- `--entrypoint` 옵션으로 ENTRYPOINT를 명시적으로 변경할 수 있다.

위 예시는 ENTRYPOINT와 CMD를 조합한 개념 설명용 구성이다.

실습에서는 Nginx 공식 이미지의 `/docker-entrypoint.sh`를 상속하여 사용했다.

### 핵심 차이

- RUN = 이미지 제작 중 실행
- CMD = 컨테이너 기본 실행 명령
- ENTRYPOINT = 컨테이너 실행 진입점

실무에서는 프로세스 종료 신호 처리를 고려하여 Exec 형식의 CMD/ENTRYPOINT 사용을 권장한다.

---

## 2. ARG와 ENV 차이

### ARG

```dockerfile
ARG BUILD_VERSION=v2
```

- 이미지 빌드 과정에서 사용할 인자를 선언한다.
- `docker build --build-arg`로 값을 전달할 수 있다.
- ARG 자체는 기본적으로 컨테이너 환경변수로 유지되지 않는다.

### ENV

```dockerfile
ENV APP_VERSION=${BUILD_VERSION}
```

- 이미지에 환경변수를 설정한다.
- 해당 이미지로 실행한 컨테이너에도 기본적으로 유지된다.
- 컨테이너 실행 시 환경변수 값을 변경할 수도 있다.

### 비교

| 구분 | ARG | ENV |
|---|---|---|
| 목적 | 빌드 시 변수 | 이미지·컨테이너 환경변수 |
| 선언 | ARG | ENV |
| 빌드 인자로 전달 | 가능 | 직접 전달 방식은 아님 |
| 컨테이너 환경변수로 기본 유지 | X | O |

### 실무 사례

CI/CD 환경에서 이미지 빌드 버전을 ARG로 전달하고, 애플리케이션에서 참조할 버전 정보를 ENV로 저장할 수 있다.

주의: ARG와 ENV는 비밀번호 등 비밀정보를 안전하게 저장하는 수단이 아니다.

---

## 3. .dockerignore

Docker 이미지 빌드 시 불필요한 파일을 Build Context에서 제외하는 파일이다.

### 예시

```text
.git
.env
*.log
*.bak
backup/
tmp/
```

### 사용하는 이유

1. 불필요한 파일을 빌드 컨텍스트에서 제외한다.
2. 빌드에 전달하는 데이터의 양을 줄인다.
3. 임시 파일과 로그가 이미지에 포함될 위험을 줄인다.
4. 빌드 효율과 관리 편의성을 높인다.

### 핵심

`COPY`는 Build Context에 포함된 파일을 대상으로 한다.

.dockerignore에 의해 제외된 파일은 일반적인 COPY로 복사할 수 없다.

단, .dockerignore만으로 완전한 비밀정보 보호가 보장되는 것은 아니다.

---

## 4. Docker Build Cache

Build Cache는 이전 이미지 빌드 결과를 재사용하는 기능이다.

### 등장 배경

Dockerfile을 빌드할 때 변경되지 않은 작업까지 매번 다시 수행하면 시간이 오래 걸릴 수 있다.

Docker는 재사용 가능한 빌드 결과를 캐시에 저장하여 불필요한 반복 작업을 줄인다.

### 동작 흐름

첫 번째 빌드
→ 각 단계 실행
→ 빌드 결과 저장

두 번째 빌드(변경 없음)
→ 기존 캐시 재사용 가능
→ CACHED 표시

파일 변경 후 빌드
→ 변경된 파일에 의존하는 단계의 캐시 무효화
→ 해당 단계와 이후 의존 단계 재처리

### 실습 결과

index.html을 수정하기 전에는 동일한 입력을 사용하는 빌드 단계에서 캐시가 재사용될 수 있었다.

index.html을 수정한 뒤에는 COPY 단계부터 관련 빌드 작업이 다시 처리될 수 있음을 학습했다.

### 핵심

- Build Cache = 이전 빌드 결과 재사용
- Cache Invalidation = 빌드 입력 변경으로 캐시 재사용이 불가능해지는 현상
- Dockerfile 명령어 순서와 파일 변경 범위는 빌드 효율에 영향을 준다.

---

## 5. 이미지 태그와 이미지 ID 차이

### Image Tag

이미지를 참조하기 위한 이름이다.

예시:

`marble-nginx:v2`

태그는 새로운 이미지를 빌드하면서 다른 이미지 ID를 가리킬 수 있다.

### Image ID

이미지의 콘텐츠와 설정에 기반한 식별자이다.

### 중요한 차이

같은 태그로 이미지를 다시 빌드해도 기존 컨테이너는 자동으로 변경되지 않는다.

### 실습 예시

1. marble-nginx:v2 이미지 생성
2. marble-image-v2 컨테이너 실행
3. index.html 수정
4. 같은 marble-nginx:v2 태그로 재빌드
5. 새로운 이미지 ID 생성
6. 기존 컨테이너는 이전 HTML 유지
7. 신규 컨테이너는 수정된 HTML 출력

### 핵심

Image Tag는 이미지 참조 이름이고, 실행 중인 컨테이너는 생성 당시 사용한 이미지에 기반한다.

---

## 6. Rebuild와 Redeploy 차이

| 개념 | 의미 |
|---|---|
| Rebuild | Docker 이미지를 다시 제작 |
| Redeploy | 제작된 이미지를 실제 서비스에 적용 |

### Rebuild

Dockerfile 또는 애플리케이션 파일을 변경한 뒤 이미지를 다시 빌드한다.

### Redeploy

새 이미지로 컨테이너를 재생성하거나 새 컨테이너를 배포하여 변경 사항을 서비스에 적용한다.

### 핵심

이미지 재빌드 ≠ 기존 컨테이너 자동 업데이트

### 실무 사례

Jenkins에서 새로운 Docker 이미지를 빌드하더라도 기존 WAS 컨테이너가 자동으로 새 버전으로 변경되지는 않는다.

새 이미지를 실제 서비스에 반영하기 위한 배포 과정이 필요하다.

---

## 7. 4장 2교시 실습 결과

### 프로젝트 구성

| 항목 | 값 |
|---|---|
| 작업 디렉터리 | ~/docker-lab/ch04-image |
| 기존 이미지 | marble-nginx:v1 |
| 신규 이미지 | marble-nginx:v2 |
| 기존 컨테이너 | marble-image-v1 |
| 신규 컨테이너 | marble-image-v2 |
| v1 호스트 포트 | 18085 |
| v2 호스트 포트 | 18086 |
| 재빌드 비교 포트 | 18087 |

### 확인한 내용

- .dockerignore 작성
- ARG 및 ENV 환경변수 설정
- CMD와 ENTRYPOINT 동작 차이 확인
- marble-nginx:v2 이미지 빌드
- v1·v2 웹페이지 비교
- Build Cache 재사용 확인
- HTML 수정 후 캐시 무효화 확인
- 기존 컨테이너와 새 이미지 ID 비교
- 신규 컨테이너에서 변경된 HTML 확인
- 실습용 신규 컨테이너 정리

---

## 8. 주요 장애 대응

### ① 이미지 재빌드 후 웹페이지가 바뀌지 않음

원인:
- 기존 컨테이너가 이전 이미지로 실행 중

해결:
- 이미지 ID 비교
- 새 이미지로 컨테이너 재생성
- HTTP 응답으로 변경 사항 검증

### ② ARG 값이 컨테이너에서 확인되지 않음

원인:
- ARG는 기본적으로 컨테이너 환경변수로 유지되지 않음

해결:
- 필요한 경우 ENV에 값을 저장
- 실제 컨테이너 환경변수 확인

### ③ COPY 파일을 찾지 못함

원인:
- Build Context 경로 오류
- 파일명 오류
- .dockerignore에 의한 제외

해결:
- 현재 디렉터리 및 파일 확인
- Dockerfile과 .dockerignore 내용 확인

### ④ 컨테이너 시작 후 바로 종료됨

원인:
- CMD 또는 ENTRYPOINT 명령 오류
- 애플리케이션 실행 실패

해결:
- docker ps -a로 상태 확인
- docker logs로 오류 확인
- docker inspect로 종료 코드 확인

---

## 9. 확인문제 정답

| 번호 | 정답 |
|---|---|
| 1 | ENTRYPOINT = 실행 진입점 |
| 2 | RUN = 빌드 중 실행, CMD = 기본 실행 명령 |
| 3 | ARG = 빌드 인자, ENV = 환경변수 |
| 4 | --build-arg = 빌드 인자 전달 |
| 5 | .dockerignore = 불필요한 파일 제외 |
| 6 | docker run에서 기본 CMD 대체 가능 |
| 7 | 파일 변경 시 관련 캐시 무효화 |
| 8 | 이미지 재빌드 후 기존 컨테이너 유지 |
| 9 | 새 이미지 적용 시 컨테이너 재생성 |
| 10 | ARG·ENV에 비밀번호 저장 금지 |

결과: 10문제 모두 정답 (100점)

---

## 10. 면접 핵심 질문

### Q1. CMD와 ENTRYPOINT의 차이는?

CMD는 컨테이너의 기본 실행 명령이나 인자를 지정하고, ENTRYPOINT는 실행할 프로그램을 지정한다.

두 명령어를 조합하여 사용할 수 있으며, docker run에서 전달한 명령과 옵션에 따라 동작을 변경할 수 있다.

### Q2. ARG와 ENV의 차이는?

ARG는 이미지 빌드 과정에서 사용할 인자이고 ENV는 이미지에 환경변수를 설정하여 컨테이너에도 기본적으로 전달한다.

### Q3. Build Cache는 왜 필요한가요?

변경되지 않은 빌드 결과를 재사용하여 불필요한 반복 작업을 줄이고 빌드 시간을 단축하기 위해서이다.

### Q4. 이미지를 다시 빌드했는데 기존 컨테이너가 변경되지 않는 이유는?

컨테이너는 생성 당시 사용한 이미지에 기반하여 실행되기 때문이다.

이미지가 새로 빌드되어도 기존 컨테이너에 자동 적용되지 않으므로 일반적으로 새 이미지로 재생성해야 한다.

### Q5. .dockerignore는 왜 사용하나요?

불필요한 파일을 Build Context에서 제외하여 빌드 효율을 높이고, 의도하지 않은 파일이 이미지 빌드에 포함될 위험을 줄이기 위해서이다.

---

## 11. 핵심 암기

- RUN = 빌드 시 명령 실행
- CMD = 컨테이너 기본 실행 명령
- ENTRYPOINT = 컨테이너 실행 진입점
- ARG = 빌드 변수
- ENV = 컨테이너 환경변수
- .dockerignore = 불필요한 빌드 파일 제외
- Build Cache = 이전 빌드 결과 재사용
- Image Tag = 이미지를 참조하는 이름
- Image ID = 이미지 식별자
- Rebuild = 이미지 다시 제작
- Redeploy = 새 이미지 실제 배포

### 최종 핵심

**Docker 이미지 빌드와 컨테이너 배포는 별개의 과정이다.**

이미지를 수정하고 재빌드해도 기존 컨테이너가 자동으로 변경되지는 않는다.

---

# 12. 핵심 명령어 및 실습 명령어 모음

## A. 반드시 익혀둘 명령어

| 명령어 | 목적 |
|---|---|
| docker build -t | 이미지 빌드 |
| docker run -d | 컨테이너 백그라운드 실행 |
| docker image ls | 이미지 목록 확인 |
| docker ps -a | 전체 컨테이너 확인 |
| docker logs | 컨테이너 로그 확인 |
| docker inspect | 컨테이너 상세 정보 확인 |
| docker stop | 컨테이너 중지 |
| docker rm | 컨테이너 삭제 |

## B. 용도를 이해하고 필요할 때 참고할 명령어

| 명령어·옵션 | 목적 |
|---|---|
| --build-arg | 빌드 인자 전달 |
| --progress=plain | 빌드 과정 상세 출력 |
| --entrypoint | 컨테이너 ENTRYPOINT 변경 |
| --format | 원하는 정보만 출력 |
| docker image inspect | 이미지 설정 및 ID 조회 |
| --rm | 컨테이너 종료 시 자동 삭제 |

## C. 실제 실습 명령어 — 작업 순서별

※ 이미 실습을 완료했으므로 아래는 복습용 기록이다. 기존 컨테이너가 있는 상태에서 동일한 docker run 명령을 재실행하면 이름 충돌이 발생할 수 있다.

### ① 실습 디렉터리 이동 및 백업

```bash
cd ~/docker-lab/ch04-image

mkdir -p ../ch04-backup

cp Dockerfile ../ch04-backup/Dockerfile.v1
cp index.html ../ch04-backup/index.v1.html

sudo docker image ls marble-nginx
sudo docker ps -a
```

### ② .dockerignore 작성

```bash
nano .dockerignore
```

파일 내용:

```text
.git
.env
*.log
*.bak
backup/
tmp/
```

### ③ HTML v2 작성

```bash
nano index.html
```

파일 내용:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Marble Docker Lab v2</title>
</head>
<body>
    <h1>Marble Docker Image v2</h1>
    <p>Dockerfile 심화 실습입니다.</p>
    <p>ARG, ENV, CMD, ENTRYPOINT 학습 중!</p>
</body>
</html>
```

### ④ Dockerfile v2 작성

```bash
nano Dockerfile
```

파일 내용:

```dockerfile
FROM nginx:alpine

ARG BUILD_VERSION=v2

ENV APP_VERSION=${BUILD_VERSION}

WORKDIR /usr/share/nginx/html

COPY index.html ./index.html

RUN test -s index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### ⑤ v2 이미지 빌드

```bash
sudo docker build --progress=plain \
  --build-arg BUILD_VERSION=v2 \
  -t marble-nginx:v2 .
```

### ⑥ 이미지와 환경변수 확인

```bash
sudo docker image ls marble-nginx

sudo docker image inspect \
  --format '{{range .Config.Env}}{{println .}}{{end}}' \
  marble-nginx:v2 | grep '^APP_VERSION='
```

### ⑦ v2 컨테이너 생성 및 실행

```bash
sudo docker run -d \
  --name marble-image-v2 \
  -p 127.0.0.1:18086:80 \
  marble-nginx:v2

sudo docker ps

curl http://127.0.0.1:18085
curl http://127.0.0.1:18086

sudo docker exec marble-image-v2 \
  printenv APP_VERSION
```

### ⑧ ENTRYPOINT와 CMD 확인

```bash
sudo docker image inspect \
  --format 'ENTRYPOINT={{json .Config.Entrypoint}} CMD={{json .Config.Cmd}}' \
  marble-nginx:v2

# 기본 CMD 대체
sudo docker run --rm marble-nginx:v2 nginx -v

# ENTRYPOINT 변경
sudo docker run --rm \
  --entrypoint /bin/sh \
  marble-nginx:v2 \
  -c 'echo ENTRYPOINT_OVERRIDE_OK'
```

### ⑨ Build Cache 확인

```bash
# 파일 변경 없이 동일하게 재빌드
sudo docker build --progress=plain \
  --build-arg BUILD_VERSION=v2 \
  -t marble-nginx:v2 .
```

빌드 출력에서 CACHED 사용 여부를 확인한다.

### ⑩ HTML 변경 후 이미지 재빌드

```bash
nano index.html
```

body 안에 다음 내용 추가:

```html
<p>Build Cache 변경 테스트 성공!</p>
```

다시 빌드:

```bash
sudo docker build --progress=plain \
  --build-arg BUILD_VERSION=v2 \
  -t marble-nginx:v2 .
```

### ⑪ 기존 컨테이너와 이미지 ID 비교

```bash
# 현재 태그가 가리키는 이미지 ID
sudo docker image inspect \
  --format '{{.Id}}' \
  marble-nginx:v2

# 기존 컨테이너가 사용하는 이미지 ID
sudo docker inspect \
  --format '{{.Image}}' \
  marble-image-v2

# 기존 컨테이너의 웹페이지 확인
curl http://127.0.0.1:18086
```

### ⑫ 새 이미지로 신규 컨테이너 실행

```bash
sudo docker run -d \
  --name marble-image-v2-new \
  -p 127.0.0.1:18087:80 \
  marble-nginx:v2

# 기존 컨테이너
curl http://127.0.0.1:18086

# 신규 컨테이너
curl http://127.0.0.1:18087

# 컨테이너별 이미지 ID
sudo docker inspect \
  --format '{{.Name}} {{.Image}}' \
  marble-image-v2 marble-image-v2-new
```

### ⑬ 실습용 신규 컨테이너 정리

```bash
sudo docker stop marble-image-v2-new
sudo docker rm marble-image-v2-new
```

### ⑭ 최종 확인

```bash
sudo docker image ls marble-nginx

sudo docker ps -a

curl http://127.0.0.1:18085
curl http://127.0.0.1:18086

sudo docker image inspect \
  --format '{{.Id}}' marble-nginx:v2

sudo docker inspect \
  --format '{{.Image}}' marble-image-v2
```
<img width="1312" height="1199" alt="image" src="https://github.com/user-attachments/assets/62fec747-8c2d-4c23-863d-34e19a6191fa" />

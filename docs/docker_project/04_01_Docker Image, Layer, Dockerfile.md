
# 4장 1교시 — Docker Image, Layer, Dockerfile

## 1. Dockerfile, Image, Container

| 구분 | 설명 |
|---|---|
| Dockerfile | Docker 이미지를 제작하기 위한 설계서 |
| Docker Image | 애플리케이션 실행에 필요한 파일과 설정을 담은 결과물 |
| Docker Container | 이미지를 기반으로 생성된 실행 인스턴스 |

### 동작 흐름

Dockerfile 작성
      ↓
docker build
      ↓
Docker Image 생성
      ↓
docker run
      ↓
Container 생성 및 실행

### 핵심 개념

- Dockerfile로 이미지 제작 과정을 정의한다.
- `docker build`로 이미지를 생성한다.
- `docker run`으로 이미지 기반 컨테이너를 실행한다.
- 하나의 이미지로 여러 컨테이너를 생성할 수 있다.
- 이미지를 재빌드해도 기존 컨테이너는 자동으로 교체되지 않는다.

---

## 2. Docker Image Layer

Docker 이미지는 여러 개의 읽기 전용 파일시스템 계층으로 구성된다.

### 등장 배경

이미지를 제작할 때 공통 파일을 매번 중복 저장하면 저장 공간과 전송 비용이 증가할 수 있다.

Docker는 공통 Layer를 재사용하여 이미지 저장과 배포를 효율적으로 처리한다.

### Layer 구조

Container Writable Layer (읽기·쓰기 가능)
                 ↓
Custom Image Layer (index.html)
                 ↓
Nginx Base Image Layers
                 ↓
Linux 기반 파일시스템 Layer

### 주요 특징

1. 이미지 파일시스템 Layer는 읽기 전용이다.
2. 공통 Layer를 여러 이미지가 재사용할 수 있다.
3. 컨테이너 실행 시 별도의 쓰기 계층이 사용된다.
4. Dockerfile의 모든 명령어가 반드시 새로운 파일시스템 Layer를 생성하는 것은 아니다.
5. 컨테이너 삭제 시 일반적으로 컨테이너 쓰기 계층도 삭제된다.

### 주요 명령어

- `docker image history`: 이미지 빌드 이력 확인
- `docker image inspect`: 이미지 상세 정보 확인

---

## 3. Dockerfile 주요 명령어

| 명령어 | 역할 |
|---|---|
| FROM | 기반 이미지 지정 |
| WORKDIR | 작업 디렉터리 설정 |
| COPY | 파일을 이미지에 복사 |
| RUN | 이미지 빌드 과정에서 명령 실행 |
| EXPOSE | 애플리케이션이 사용하는 포트 정보 기록 |
| CMD | 컨테이너 기본 실행 명령 지정 |

### Dockerfile 작성 예시

```dockerfile
# 기반 이미지
FROM nginx:alpine

# 작업 디렉터리
WORKDIR /usr/share/nginx/html

# 웹페이지 복사
COPY index.html ./index.html

# 웹페이지 파일 검증
RUN test -s index.html

# 컨테이너 사용 포트 명시
EXPOSE 80

# Nginx 기본 실행 명령
CMD ["nginx", "-g", "daemon off;"]
```

### 주요 개념

**FROM**
- 기존 이미지를 기반으로 새로운 이미지를 제작한다.

**WORKDIR**
- 이후 실행되는 명령어의 작업 디렉터리를 지정한다.

**COPY**
- Build Context에 있는 파일을 이미지 내부로 복사한다.

**RUN**
- 이미지 빌드 과정에서 명령어를 실행한다.
- 이번 실습에서는 HTML 파일이 비어 있지 않은지 검증했다.

**EXPOSE**
- 컨테이너가 사용하는 포트를 문서화한다.
- EXPOSE만으로 호스트 포트가 자동 게시되지는 않는다.

**CMD**
- 컨테이너를 시작할 때 사용할 기본 명령어를 지정한다.
- 이번 실습에서는 Nginx를 포그라운드에서 실행하도록 지정했다.

### RUN과 CMD의 차이

| RUN | CMD |
|---|---|
| 빌드 시 실행 | 컨테이너 시작 시 사용 |
| 설치·파일 생성·검증 | 애플리케이션 실행 |
| 빌드 결과에 영향 | 실행 설정에 영향 |

---

## 4. Docker Build

Dockerfile을 기반으로 이미지를 제작하는 작업이다.

### 기본 문법

`docker build -t 이미지명:태그 빌드컨텍스트`

### 옵션

| 옵션 | 의미 |
|---|---|
| build | 이미지 빌드 |
| -t | 이미지 이름 및 태그 지정 |
| --progress=plain | 빌드 진행 과정을 텍스트로 출력 |
| . | 현재 디렉터리를 Build Context로 지정 |

### Build Context

이미지 빌드 과정에서 Docker가 사용할 수 있도록 제공하는 파일 집합이다.

예를 들어 `COPY index.html ./index.html`은 Build Context에 포함된 index.html 파일을 복사한다.

Build Context에 파일이 없거나 .dockerignore에서 제외되면 COPY 작업이 실패할 수 있다.

---

## 5. Docker Build Cache

Docker는 이전 빌드 단계의 결과를 재사용할 수 있다.

### 필요한 이유

- 변경되지 않은 빌드 결과 재사용
- 불필요한 작업 반복 감소
- 이미지 빌드 시간 단축

### 동작 방식

첫 번째 빌드:
각 단계 실행 → 빌드 결과 생성

두 번째 빌드:
변경 사항 없음 → 캐시 재사용 가능

파일 변경:
해당 파일에 의존하는 빌드 단계의 캐시가 무효화될 수 있음

### 핵심

빌드 로그에 `CACHED`가 나타나면 이전 빌드 결과가 재사용된 것이다.

Dockerfile의 명령어 순서와 파일 변경 범위는 캐시 효율에 영향을 준다.

---

## 6. 이미지와 컨테이너의 관계

이미지를 새로 빌드해도 기존 컨테이너가 자동으로 새 이미지로 변경되는 것은 아니다.

### 예시

1. marble-nginx:v1 이미지 빌드
2. 해당 이미지로 marble-image-v1 컨테이너 실행
3. HTML 수정 후 동일한 태그로 이미지 재빌드
4. 기존 컨테이너는 이전 이미지 기반으로 계속 실행
5. 변경 사항을 적용하려면 새 이미지로 컨테이너 재생성 필요

### 핵심

- Image Build ≠ Container Update
- Image Tag ≠ 실행 중인 컨테이너 자동 업데이트
- 새 이미지 적용을 위해서는 일반적으로 컨테이너 재생성이 필요하다.

---

## 7. 실습 결과

### 제작한 이미지

- 이미지: marble-nginx:v1
- 기반 이미지: nginx:alpine
- 컨테이너: marble-image-v1
- 호스트 포트: 127.0.0.1:18085
- 컨테이너 포트: 80

### 검증 결과

- Dockerfile 작성 완료
- Docker 이미지 빌드 성공
- Nginx 컨테이너 실행 성공
- 사용자 정의 HTML 페이지 출력 성공
- Build Cache 및 Layer 확인 학습

실제 HTTP 응답 본문:

```html
<h1>Marble Docker Image v1</h1>
<p>Dockerfile로 직접 제작한 Nginx 이미지입니다.</p>
```

---

## 8. 주요 장애 대응

### ① Dockerfile을 찾을 수 없는 경우

원인:
- 작업 디렉터리 오류
- Dockerfile 파일명 오류

해결:
- pwd로 현재 위치 확인
- ls -la로 Dockerfile 존재 여부 확인

### ② COPY 파일을 찾을 수 없는 경우

원인:
- Build Context에 필요한 파일이 없음
- 파일명·경로가 잘못됨
- .dockerignore에 의해 파일이 제외됨

해결:
- 파일 경로와 Build Context 확인

### ③ 기본 Nginx 페이지만 출력되는 경우

원인:
- 사용자 정의 HTML이 올바른 위치에 복사되지 않음
- 기존 컨테이너가 이전 이미지로 실행 중일 수 있음

해결:
- Dockerfile의 COPY 경로 확인
- 컨테이너 내부 HTML 확인
- 이미지 ID 비교 및 필요 시 컨테이너 재생성

### ④ 포트 충돌

원인:
- 다른 프로세스 또는 컨테이너가 동일한 호스트 포트를 사용함

해결:
- 호스트 포트 사용 상태 확인
- 충돌하지 않는 포트 사용

---

## 9. 확인문제 정답

| 번호 | 정답 |
|---|---|
| 1 | Docker 이미지 제작 방법 정의 |
| 2 | docker build |
| 3 | FROM = 기반 이미지 지정 |
| 4 | RUN = 빌드 시 실행, CMD = 기본 실행 명령 |
| 5 | COPY = 파일 복사 |
| 6 | EXPOSE = 사용 포트 정보 기록 |
| 7 | . = 현재 디렉터리 Build Context |
| 8 | 공통 Layer 재사용 가능 |
| 9 | Build Cache = 이전 빌드 결과 재사용 |
| 10 | 이미지 재빌드 시 기존 컨테이너 자동 교체되지 않음 |

결과: 10문제 전부 정답 (100점)

---

## 10. 면접 핵심 질문

### Q1. Dockerfile이란 무엇인가요?

Docker 이미지 제작 과정을 정의하는 파일입니다.

FROM으로 기반 이미지를 지정하고 COPY, RUN 등의 명령어를 사용하여 필요한 파일과 환경을 구성할 수 있습니다.

### Q2. Docker Image Layer란 무엇인가요?

Docker 이미지의 파일시스템을 구성하는 읽기 전용 계층입니다.

공통 Layer를 재사용할 수 있어 저장 공간과 이미지 배포의 효율을 높일 수 있습니다.

### Q3. RUN과 CMD의 차이는 무엇인가요?

RUN은 이미지 빌드 과정에서 명령어를 실행하고, CMD는 컨테이너를 시작할 때 사용할 기본 명령어를 지정합니다.

### Q4. Build Cache를 사용하는 이유는?

변경되지 않은 빌드 결과를 재사용하여 이미지 빌드 시간을 단축하기 위해서입니다.

### Q5. 이미지를 다시 빌드하면 기존 컨테이너에도 변경 사항이 적용되나요?

아닙니다.

기존 컨테이너는 생성할 당시 사용한 이미지를 기반으로 실행됩니다. 새로운 이미지의 변경 사항을 적용하려면 일반적으로 컨테이너를 재생성해야 합니다.

---

## 11. 핵심 암기

- Dockerfile = 이미지 제작 설계서
- Image = 컨테이너 실행에 필요한 결과물
- Container = 이미지 기반 실행 인스턴스
- FROM = 기반 이미지
- WORKDIR = 작업 디렉터리
- COPY = 파일 복사
- RUN = 빌드 중 명령 실행
- EXPOSE = 사용 포트 명시
- CMD = 기본 실행 명령
- docker build = 이미지 제작
- Layer = 재사용 가능한 이미지 계층
- Build Cache = 이전 빌드 결과 재사용

**최종 핵심**

Dockerfile 작성 → Image Build → Container Run → HTTP 검증

이미지를 재빌드해도 기존 실행 컨테이너는 자동으로 변경되지 않는다.
<img width="1312" height="1199" alt="image" src="https://github.com/user-attachments/assets/97d0fec2-6c1e-42f2-850a-139be5155e86" />

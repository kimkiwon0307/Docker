<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/7219ee97-32cf-4461-9073-6cdf398fd3ad" />


# Docker 5장 1교시 — Multi-stage Build 및 이미지 최적화

## 1. Multi-stage Build란?

하나의 Dockerfile에서 **빌드 환경(Builder)**과 **실행 환경(Runtime)**을 분리하여 최종 Docker 이미지를 최적화하는 방법이다.

### 사용하는 이유
- 불필요한 빌드 도구 제거로 이미지 크기 감소
- 이미지 다운로드 및 배포 효율 개선
- 불필요한 프로그램을 줄여 보안 위험 감소
- 빌드 환경과 실행 환경 분리

## 2. Multi-stage Build 동작 원리

```text
[Builder]
JDK + Gradle + Java 소스코드
        ↓
    gradle bootJar
        ↓
      app.jar
        ↓
  COPY --from=builder
        ↓
[Runtime]
JRE + app.jar
        ↓
Spring Boot 컨테이너 실행
```

- **Builder:** 소스코드를 컴파일하고 실행 파일 생성
- **Runtime:** 완성된 실행 파일만 가져와 애플리케이션 실행
- **JDK:** Java 개발 및 컴파일 환경
- **JRE:** Java 실행 환경
- **Gradle:** Java 프로젝트 빌드 도구
- **JAR:** Java 애플리케이션 패키지

## 3. Dockerfile 핵심 문법

| 명령어 | 역할 |
|---|---|
| `FROM ... AS builder` | 빌드 단계 생성 및 이름 지정 |
| `RUN` | 이미지 빌드 중 명령 실행 |
| `FROM ... AS runtime` | 실행 단계 생성 |
| `COPY --from=builder` | 이전 단계에서 필요한 파일 복사 |
| `ENTRYPOINT` | 컨테이너 실행 진입점 지정 |

### Spring Boot Multi-stage Dockerfile 예시

```dockerfile
# 1단계: Builder
FROM gradle:8.14.3-jdk17 AS builder
WORKDIR /app
COPY --chown=gradle:gradle . .
USER gradle
RUN gradle --no-daemon bootJar

# 2단계: Runtime
FROM eclipse-temurin:17-jre AS runtime
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

※ 위 코드는 Multi-stage 구조를 이해하기 위한 예시이며, 실제 프로젝트에서는 JAR 파일명과 권한, 빌드 캐시 등을 확인해야 한다.

## 4. 추가 이미지 최적화 방법

| 방법 | 목적 |
|---|---|
| Multi-stage Build | 빌드 도구와 실행 환경 분리 |
| 경량 Runtime 이미지 | 불필요한 구성요소 감소 |
| `.dockerignore` | 빌드에 필요 없는 파일 제외 |
| Build Cache | 변경되지 않은 빌드 결과 재사용 |
| Non-root 실행 | 컨테이너 실행 권한 최소화 |

**Build Cache 최적화:** 변경이 적은 의존성 준비 단계를 먼저 배치하면 소스코드 수정 시 캐시 재사용에 유리하다.

## 5. 핵심 명령어

```bash
# 이미지 빌드
sudo docker build -t marble-multistage:v1 .

# 이미지 목록 및 크기 확인
sudo docker image ls

# 이미지 빌드 이력 확인
sudo docker image history marble-multistage:v1

# 컨테이너 실행
sudo docker run -d \
  --name marble-multistage-review \
  -p 127.0.0.1:18096:80 \
  marble-multistage:v1

# 실행 상태 확인
sudo docker ps

# HTTP 응답 확인 (Nginx 실습 예제)
curl http://127.0.0.1:18096
```

## 6. 핵심 요약

- **Multi-stage Build = Builder와 Runtime 분리**
- `FROM ... AS` = 빌드 단계 이름 지정
- `COPY --from` = 다른 단계에서 필요한 파일만 복사
- 최종 이미지에서 빌드 도구를 제외해 이미지 크기와 공격 표면을 줄일 수 있다.
- Multi-stage Build가 CPU·메모리 사용량까지 자동으로 줄이는 것은 아니다.

**학습 결과:** 5장 1교시 개념 학습 완료 / 객관식 8문제 중 8문제 정답 (100%)

**다음 학습:** 6장 1교시 — Docker Network·DNS·컨테이너 간 통신

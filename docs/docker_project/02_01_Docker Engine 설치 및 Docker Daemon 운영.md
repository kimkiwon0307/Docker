
# 2장 1교시 — Docker Engine 설치 및 Docker Daemon 운영

## 1. Docker Engine이란?

Docker Engine은 Docker 이미지, 컨테이너, 네트워크, 볼륨 등을 생성하고 관리하는 실행 환경이다.

### 주요 구성요소

| 구성요소 | 역할 |
|---|---|
| Docker Client | 사용자 명령어를 Docker Daemon에 전달 |
| Docker Daemon (dockerd) | 컨테이너·이미지·네트워크 등 관리 |
| containerd | 컨테이너 생명주기 관리 |
| runc | Linux 컨테이너 생성 및 실행 |
| Docker Socket | Client와 Daemon 사이의 로컬 통신 경로 |

### Docker 동작 흐름

사용자 → Docker Client → Docker Daemon → containerd/runc → Linux Kernel → 컨테이너 실행

- Docker Client는 명령어를 API 요청으로 전달한다.
- Docker Daemon은 실제 컨테이너 관리 작업을 수행한다.
- Linux 커널의 Namespace와 Cgroups 등의 기능을 활용한다.
- 기본 로컬 통신 경로는 `/var/run/docker.sock`이다.

## 2. Docker 설치 방식

Ubuntu에서 Docker를 설치하는 대표적인 방법

1. Ubuntu 기본 저장소의 docker.io 패키지
2. Docker 공식 APT 저장소
3. Docker 설치 스크립트
4. 수동 DEB 패키지 설치

### 공식 APT 저장소를 선택한 이유

- Docker 공식 패키지를 설치할 수 있다.
- APT로 버전 및 업데이트를 관리할 수 있다.
- Docker Compose와 Buildx 플러그인을 함께 설치할 수 있다.

## 3. Docker Engine 설치 실습

환경: Ubuntu Server 24.04 LTS

### ① 패키지 및 GPG 공개키 준비

```bash
sudo apt update
sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL \
  https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

- `apt update`: 패키지 목록 갱신
- `-y`: 설치 확인에 자동 동의
- `install -d`: 디렉터리 생성
- `-m 0755`: 디렉터리 권한 설정
- `curl -fsSL`: HTTP 오류 확인, 진행 출력 최소화, 리다이렉트 처리
- `-o`: 다운로드 파일 경로 지정
- `chmod a+r`: 모든 사용자에게 읽기 권한 부여

GPG 공개키는 APT 저장소의 패키지 메타데이터 서명을 검증하는 데 사용한다.

### ② Docker 공식 저장소 등록

```bash
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

### ③ Docker 패키지 설치

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

| 패키지 | 역할 |
|---|---|
| docker-ce | Docker Engine 서버 |
| docker-ce-cli | Docker CLI |
| containerd.io | 컨테이너 런타임 |
| docker-buildx-plugin | 이미지 빌드 기능 |
| docker-compose-plugin | 멀티 컨테이너 관리 |

## 4. Docker 서비스 관리

Docker Daemon은 Ubuntu에서 systemd 서비스로 관리할 수 있다.

### 주요 명령어

```bash
# Docker 서비스 상태
sudo systemctl status docker

# Docker 서비스 시작
sudo systemctl start docker

# Docker 서비스 중지
sudo systemctl stop docker

# Docker 서비스 재시작
sudo systemctl restart docker

# 부팅 시 자동 실행 및 즉시 시작
sudo systemctl enable --now docker

# Docker 서비스 로그 최근 30줄
sudo journalctl -u docker -n 30 --no-pager
```

### 핵심 개념

- `systemctl`: systemd 서비스 관리
- `enable`: 부팅 시 자동 시작 설정
- `--now`: 설정과 동시에 즉시 실행
- `journalctl`: 서비스 로그 조회
- `-u docker`: Docker 서비스 로그 선택
- `-n 30`: 최근 30줄 출력

정상 서비스 상태: `active (running)`

운영 중 Docker 서비스를 중지하거나 재시작하면 컨테이너에 영향을 줄 수 있으므로 주의한다.

## 5. Docker 설치 및 컨테이너 실행 검증

```bash
# Docker 버전 및 Client/Server 확인
sudo docker version

# Docker 서비스 실행 확인
sudo systemctl is-active docker

# 첫 번째 컨테이너 실행
sudo docker run hello-world

# 실행 중인 컨테이너 목록
sudo docker ps

# 종료된 컨테이너까지 확인
sudo docker ps -a

# 다운로드된 이미지 확인
sudo docker image ls
```

### hello-world 실행 흐름

1. Docker Client가 실행 요청
2. Docker Daemon이 로컬 이미지 확인
3. 이미지가 없으면 Docker Hub에서 다운로드
4. 이미지 기반으로 컨테이너 생성 및 실행
5. 메시지 출력 후 컨테이너 정상 종료

성공 메시지: `Hello from Docker!`

### docker ps와 docker ps -a의 차이

| 명령어 | 기능 |
|---|---|
| docker ps | 실행 중인 컨테이너만 조회 |
| docker ps -a | 종료된 컨테이너까지 조회 |
| docker image ls | 로컬 Docker 이미지 조회 |

hello-world는 메시지를 출력한 후 종료되므로 `docker ps`에는 나타나지 않을 수 있다.

## 6. Docker 보안

- Docker Daemon은 일반적인 기본 설치에서 root 권한으로 실행된다.
- Docker Client는 Unix Socket을 통해 Daemon과 통신한다.
- Docker 그룹에 속한 사용자는 사실상 root 수준의 권한을 얻을 수 있다.
- 따라서 Docker 그룹 접근 권한을 신중하게 관리해야 한다.
- 이번 실습에서는 `sudo docker` 방식을 사용한다.

## 7. 장애 대응

### ① Docker Daemon 연결 오류

오류:
`Cannot connect to the Docker daemon`

점검:

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 30 --no-pager
```

서비스가 중지되어 있다면 원인을 확인한 뒤 시작한다.

```bash
sudo systemctl start docker
```

### ② Docker Socket 권한 오류

오류:
`permission denied`

확인:

```bash
sudo ls -l /var/run/docker.sock
sudo docker ps
```

원인 중 하나는 현재 사용자의 Docker Socket 접근 권한 부족이다.

### ③ Docker 패키지 검색 실패

오류:
`Unable to locate package docker-ce`

점검:

```bash
cat /etc/apt/sources.list.d/docker.sources
apt-cache policy docker-ce
sudo apt update
```

저장소 등록, Ubuntu 코드명, GPG 공개키를 확인한다.

### ④ 이미지 다운로드 실패

DNS, 인터넷 연결, 레지스트리 연결 상태를 점검한다.

```bash
getent hosts registry-1.docker.io
curl -I https://registry-1.docker.io/v2/
```

레지스트리 API가 인증되지 않은 요청에 `401 Unauthorized`를 반환하는 것은 정상일 수 있다.

## 8. 확인문제 정답

1. Docker Engine에서 실행과 관리를 담당하는 서비스?
   - Docker Daemon

2. Docker Client의 역할?
   - 사용자 명령어를 Docker Daemon에 전달

3. Docker 공식 APT 저장소를 사용하는 이유?
   - 공식 패키지 설치 및 업데이트 관리

4. GPG 공개키를 사용하는 이유?
   - 패키지 저장소 정보의 서명 검증

5. Docker 자동 시작 및 즉시 실행 명령어?
   - sudo systemctl enable --now docker

6. Docker Client와 Daemon의 로컬 통신 경로?
   - /var/run/docker.sock

7. hello-world가 docker ps에 보이지 않는 이유?
   - 실행 후 정상적으로 종료되기 때문

8. Docker 그룹 사용자에게 높은 권한이 부여되는 이유?
   - Docker Socket을 통해 권한 높은 Daemon을 제어할 수 있기 때문

## 9. 면접 핵심 답변

**Q1. Docker Client와 Docker Daemon의 차이는?**

Docker Client는 사용자의 명령어를 Docker API 요청으로 전달하고, Docker Daemon은 이미지 및 컨테이너 생성·실행·관리 작업을 수행한다.

**Q2. Docker Daemon이 실행되지 않으면 어떻게 하나요?**

systemctl로 Docker 서비스 상태를 확인하고 journalctl로 로그를 분석하여 원인을 파악한다. 서비스가 중지된 경우 원인을 해결한 뒤 다시 시작한다.

**Q3. Docker 그룹 권한은 왜 주의해야 하나요?**

Docker 그룹에 속한 사용자는 Docker Socket을 통해 권한 높은 Daemon을 제어할 수 있으므로 사실상 root 수준의 권한을 얻을 수 있기 때문이다.

---

## 핵심 암기

- Docker Engine = 컨테이너 실행 및 관리 환경
- Docker Client = 사용자 명령어 전달
- Docker Daemon = 실제 관리 작업 수행
- Docker Socket = Client와 Daemon의 통신 경로
- systemd = Docker 서비스 관리
- docker run = 컨테이너 생성 및 실행
- docker ps -a = 종료된 컨테이너까지 확인

> Docker Client가 API 요청을 전달하면 Docker Daemon이 컨테이너 런타임 및 Linux 커널 기능을 활용하여 컨테이너를 실행하고 관리한다.

<img width="1055" height="1491" alt="image" src="https://github.com/user-attachments/assets/50fa798c-d63b-485b-b858-7566af10d387" />


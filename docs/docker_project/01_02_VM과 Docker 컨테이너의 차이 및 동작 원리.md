
# 1장 2교시 — VM과 Docker 컨테이너의 차이 및 동작 원리

## 1. 가상머신(VM)이란?

VM은 Hypervisor를 통해 가상 하드웨어를 제공받고, 독립된 Guest OS와 커널을 실행하는 가상 서버이다.

- **Host OS:** 물리 서버에서 실행되는 운영체제
- **Guest OS:** 가상머신 내부에 설치되는 운영체제
- **Hypervisor:** 가상머신을 생성하고 실행·관리하는 가상화 계층

### Hypervisor 종류

| 구분 | 특징 | 예시 |
|---|---|---|
| Type 1 | 하드웨어 위에서 직접 동작 | VMware ESXi |
| Type 2 | 호스트 OS 위에서 동작 | VMware Workstation, VirtualBox |

## 2. VM과 Docker 컨테이너 비교

| 구분 | VM | Docker Container |
|---|---|---|
| 가상화 방식 | 하드웨어 가상화 | OS 수준 격리 |
| 운영체제 | 독립된 Guest OS 실행 | 호스트 커널 공유 |
| 커널 | VM마다 독립 | Linux 호스트 커널 공유 |
| 시작 속도 | 상대적으로 느림 | 상대적으로 빠름 |
| 자원 사용 | 상대적으로 큼 | 상대적으로 적음 |
| 격리 | VM 단위 | 프로세스 단위 |

### Docker 컨테이너가 가벼운 이유

- VM은 각자 Guest OS와 커널을 실행한다.
- Linux 컨테이너는 호스트 Linux 커널을 공유한다.
- 별도 Guest OS와 커널을 실행하지 않아 추가 자원 사용이 상대적으로 적다.

## 3. Namespace란?

Namespace는 프로세스가 바라보는 시스템 자원을 격리하는 Linux 커널 기능이다.

| Namespace | 역할 |
|---|---|
| PID | 프로세스 번호 공간 격리 |
| NET | IP·네트워크 인터페이스·라우팅 등 격리 |
| MNT | 파일시스템 마운트 지점 격리 |
| UTS | 호스트명 등 격리 |
| IPC | 프로세스 간 통신 자원 격리 |
| USER | 사용자·그룹 ID 매핑 및 격리 |

**핵심: Namespace = 격리(Isolation)**

## 4. Cgroups란?

Cgroups(Control Groups)는 프로세스 그룹의 시스템 자원 사용을 측정하고 제어하는 Linux 커널 기능이다.

| 컨트롤러 | 역할 |
|---|---|
| CPU | CPU 사용량 제어 |
| Memory | 메모리 사용량 제한 |
| IO | 디스크 I/O 등 제어 |
| PIDs | 프로세스 수 제한 |

예시:
- WAS 컨테이너 메모리를 1GB로 제한
- CPU 사용량 제한
- 메모리 제한 초과 시 OOM으로 프로세스 종료 가능

**핵심: Cgroups = 자원 제어(Resource Control)**

## 5. Namespace와 Cgroups의 차이

| 구분 | Namespace | Cgroups |
|---|---|---|
| 목적 | 시스템 환경 격리 | 자원 사용 제어 |
| 주요 기능 | PID·네트워크·마운트 격리 | CPU·메모리·I/O 제한 |
| 핵심 키워드 | Isolation | Resource Control |

## 6. Docker Engine 구조

Docker Engine은 Docker Client, Docker Daemon, 컨테이너 런타임 등을 통해 컨테이너 실행과 관리를 지원한다.

### 주요 구성요소

| 구성요소 | 역할 |
|---|---|
| Docker Client | Docker 명령어 입력 및 API 요청 |
| Docker Daemon (dockerd) | 이미지·컨테이너·네트워크 등 관리 |
| containerd | 컨테이너 생명주기 관리 |
| runc | 저수준 컨테이너 생성·실행 |
| Linux Kernel | Namespace·Cgroups 등 커널 기능 제공 |

### 컨테이너 실행 흐름

사용자 → Docker Client → Docker Daemon
→ containerd / runc → Linux Kernel → 컨테이너 실행

※ 실제 실행 과정에는 추가 런타임 구성요소가 관여할 수 있다.

## 7. Linux 기초 실습 명령어

### 커널 및 가상화 환경 확인

```bash
uname -r
systemd-detect-virt
ps -p 1 -o pid,comm,args
```

### Namespace 확인

```bash
sudo lsns -t pid,net,mnt -o TYPE,NS,NPROCS,PID,COMMAND
readlink /proc/1/ns/pid
readlink /proc/self/ns/pid
```

### Cgroups 확인

```bash
stat -fc %T /sys/fs/cgroup
cat /sys/fs/cgroup/cgroup.controllers
cat /proc/self/cgroup
```

- `cgroup2fs` 출력: Cgroups v2 사용
- `cpu`, `memory`, `io`, `pids`: 자원 제어 컨트롤러 예시

## 8. 실무 장애 대응 연결

| 장애 상황 | 확인할 개념 |
|---|---|
| 메모리 부족으로 컨테이너 종료 | Cgroups, OOM |
| 컨테이너 간 통신 실패 | NET Namespace, Docker Network |
| 프로세스가 보이지 않음 | PID Namespace |
| 파일 경로가 다름 | MNT Namespace, 파일시스템 |
| 호스트 VM 자체 장애 | VM, 커널, 호스트 자원 |

## 9. 면접 핵심 질문 및 답변

**Q1. VM과 Docker의 차이는 무엇인가요?**

VM은 Hypervisor를 통해 독립된 Guest OS와 커널을 실행한다. 반면 Linux 컨테이너는 호스트 커널을 공유하며 프로세스를 격리하여 실행한다.

**Q2. Docker 컨테이너가 상대적으로 가벼운 이유는?**

컨테이너마다 별도의 Guest OS와 커널을 실행하지 않고 호스트 커널을 공유하기 때문이다.

**Q3. Namespace란 무엇인가요?**

프로세스, 네트워크, 마운트 등 시스템 자원을 격리하는 Linux 커널 기능이다.

**Q4. Cgroups란 무엇인가요?**

CPU, 메모리, I/O 등 프로세스 그룹의 자원 사용을 제어하고 측정하는 Linux 커널 기능이다.

**Q5. Docker는 VM 내부에서도 사용할 수 있나요?**

가능하다. 예를 들어 VMware에서 Ubuntu VM을 실행하고, 그 내부에 Docker Engine을 설치해 컨테이너를 실행할 수 있다. 이때 컨테이너는 Ubuntu VM의 Linux 커널을 공유한다.

**Q6. VM과 Docker 중 무엇이 더 좋은가요?**

무조건 우수한 기술은 없다. OS 단위의 독립성과 격리가 중요하면 VM이 적합할 수 있고, 애플리케이션 배포와 환경 표준화가 중요하면 Docker가 유리할 수 있다. 두 기술을 함께 사용할 수도 있다.

---

## 핵심 암기

- **VM:** 독립된 Guest OS와 커널 실행
- **Container:** 호스트 Linux 커널 공유
- **Namespace:** 프로세스·네트워크 등 환경 격리
- **Cgroups:** CPU·메모리 등 자원 사용 제어
- **Hypervisor:** 가상머신 실행 및 관리
- **Docker Engine:** 컨테이너 생성·실행·관리

> Docker 컨테이너는 Linux 커널의 Namespace를 활용해 실행 환경을 격리하고, Cgroups를 활용해 자원 사용을 제어할 수 있다.
<img width="1055" height="1491" alt="image" src="https://github.com/user-attachments/assets/e340d97d-2da8-4259-b25a-adb0d86dee30" />

# jenkins-docker-deploy
Jenkins 기반 Docker CI/CD 자동 배포 실습 프로젝트

### 2026-01-01

####  CI/CD Architecture (Manual Build)

<img width="1536" height="1024" alt="architecture" src="https://github.com/user-attachments/assets/8d8cb6c8-c2d1-4f57-a3f7-ef9446c87d47" />

위 아키텍처는 Jenkins를 사용하여 Docker 기반 애플리케이션을
수동으로 빌드 및 배포하는 CI/CD 파이프라인 구조를 나타냅니다.

개발자는 GitHub 저장소에 애플리케이션 소스를 관리하며,
Jenkins EC2 서버에서 직접 빌드를 실행하면
Jenkins가 소스를 체크아웃하고 Docker 이미지를 빌드합니다.

---

### 🔹 GitHub Repository
- 애플리케이션 소스 코드 관리
- Jenkinsfile, Dockerfile, 정적 웹 리소스 포함

### 🔹 Jenkins EC2 Instance
- CI/CD 파이프라인을 실행하는 빌드 서버
- GitHub 저장소에서 소스를 체크아웃
- Docker 이미지를 빌드하고 Docker Hub에 푸시
- SSH를 통해 서비스 EC2 서버로 배포 수행

### 🔹 Docker Hub
- Jenkins에서 빌드한 Docker 이미지를 저장하는 이미지 레지스트리
- 서비스 EC2 서버는 항상 최신 이미지를 pull하여 실행

### 🔹 Service EC2 Instance
- 실제 웹 서비스를 제공하는 서버
- Docker Hub에서 이미지를 pull하여 컨테이너 실행
- Nginx 컨테이너를 통해 외부 사용자 요청 처리

※ 본 단계에서는 GitHub Webhook을 사용하지 않고,
Jenkins에서 수동으로 빌드를 트리거하는 방식으로
CI/CD 파이프라인을 구성했습니다.

향후 GitHub Webhook을 적용하여
자동 배포 구조로 확장할 예정입니다.

---

### 2026-01-01

####  CI/CD Architecture (GitHub Webhook – Auto Deploy)

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/274c555a-0592-48f8-b01f-a238db4ff3a0" />

위 아키텍처는 GitHub Webhook을 활용하여  
**코드 변경 시 자동으로 빌드 및 배포가 수행되는 CI/CD 파이프라인 구조**를 나타냅니다.

개발자가 GitHub 저장소에 `git push`를 수행하면  
GitHub Webhook 이벤트가 Jenkins 서버로 전달되고,  
Jenkins는 별도의 수동 조작 없이 Pipeline을 자동 실행합니다.

---

### 🔹 GitHub Repository (Webhook Trigger)

- 애플리케이션 소스 코드 관리
- Jenkinsfile, Dockerfile, 정적 웹 리소스 포함
- `main` 브랜치 기준 배포
- GitHub Webhook 설정
  - Jenkins Webhook Endpoint:
    ```
    http://<JENKINS_PUBLIC_IP>:8080/github-webhook/
    ```
- 코드 Push 시 Jenkins Pipeline 자동 트리거

---

### 🔹 Jenkins EC2 Instance (CI Server)

- GitHub Webhook 이벤트 수신
- Pipeline Script from SCM 방식 사용
- 주요 자동 처리 단계:
  1. GitHub 저장소 코드 Checkout
  2. Docker 이미지 Build
  3. Docker Hub에 이미지 Push (`latest`)
  4. SSH를 통해 Service EC2로 자동 배포

---

### 🔹 Service EC2 Instance

- 실제 웹 서비스를 제공하는 서버
- Docker Engine 실행
- Jenkins가 SSH로 접속하여 컨테이너 자동 재배포
- 컨테이너 실행 정보:
  - Image: `yangmw7/jenkins-web:latest`
  - Port Mapping: `80:80`
- 외부 사용자는 **80 포트로 직접 접근 가능**

---

###  Auto CI/CD Test & Result

아래는 GitHub Webhook 기반 자동 배포가  
정상적으로 동작함을 확인한 실제 테스트 결과입니다.

---

####  [사진 1] GitHub Webhook 정상 전송 확인
<img width="876" height="542" alt="image" src="https://github.com/user-attachments/assets/8f1bceed-1ec3-4f91-b53a-f7df3f6ed09c" />

- GitHub Webhook `push` 이벤트 발생
- Jenkins Webhook Endpoint 응답 `200 OK`
- Webhook → Jenkins 연결 정상 동작 확인

---

####  [사진 2] Jenkins Pipeline 자동 실행 로그
<img width="862" height="957" alt="image" src="https://github.com/user-attachments/assets/07bb16c1-91f5-4b55-882e-4d29d6a58c45" />

- GitHub push 이벤트로 Pipeline 자동 시작
- Docker Build / Push / Deploy 단계 모두 SUCCESS
- 수동 개입 없는 CI/CD 자동화 확인

---

####  [사진 3] Service EC2 Docker 컨테이너 상태
<img width="1463" height="112" alt="image" src="https://github.com/user-attachments/assets/0604d2f7-66a5-4985-98aa-0a160500b019" />

- 최신 이미지 자동 pull 완료
- 컨테이너 정상 실행 확인
- 포트 매핑: 0.0.0.0:80 → 80/tcp

---

####  [사진 4] 웹 서비스 자동 반영 결과
<img width="667" height="262" alt="image" src="https://github.com/user-attachments/assets/996d763d-1b56-4f87-ac8b-d537777a7ef0" />

- `index.html` 수정 후 GitHub에 push만 수행
- Jenkins 자동 배포 후 웹 화면 즉시 변경
- 배포 시간 및 CI/CD 성공 메시지 출력

---

###  Summary

- Jenkins + Docker + GitHub Webhook 기반 자동 CI/CD 구축
- 수동 빌드 방식에서 자동 배포 구조로 확장
- 서비스 포트를 80으로 통합하여 실서비스 형태로 구성
- 실제 운영 환경과 유사한 CI/CD 파이프라인 실습 완료

---

## Next Step: CI Quality Gate 적용 (Test Failure Stops Deployment)

현재 CI/CD 파이프라인은 코드 변경 시 자동으로 배포가 수행되는 구조로 완성되었다.  
다음 단계에서는 **테스트 결과를 기준으로 배포 여부를 판단하는 CI 품질 게이트(Quality Gate)**를 적용할 예정이다.

### Planned Enhancement
- Jenkins Pipeline에 Test Stage 추가
- 테스트 실패 시 Docker Build 및 Deploy 단계 자동 중단
- 검증되지 않은 코드의 운영 환경 배포 방지

### Planned CI/CD Flow

GitHub Push
→ Jenkins Trigger
→ Test Stage (FAIL 시 Pipeline 중단)
→ Docker Build
→ Docker Push
→ Deploy

이를 통해 단순 자동 배포를 넘어  
**운영 안정성과 신뢰성을 고려한 CI/CD 파이프라인**으로 확장할 계획이다.

---

### 2026-01-02 — Elastic IP 적용 (고정 퍼블릭 IP 구성)

기존 EC2 인스턴스는 퍼블릭 IPv4 주소가 **임시(Ephemeral) IP**로 할당되어 있어,  
인스턴스를 중지(Stop) 후 다시 시작(Start)할 경우 퍼블릭 IP 주소가 변경되는 문제가 발생했다.

이로 인해 다음과 같은 문제가 발생할 수 있다.
- Jenkins 접속 주소 변경
- GitHub Webhook 엔드포인트 불일치
- Jenkins → Service EC2 SSH 배포 실패
- CI/CD 파이프라인 재설정 필요

이를 해결하기 위해 **Jenkins EC2와 Service EC2 모두에 Elastic IP를 적용**하여  
인스턴스 재시작 여부와 관계없이 **고정된 퍼블릭 IP 주소를 유지**하도록 구성하였다.

---

#### Elastic IP 적용 대상

- **Jenkins-EC2**
  - Elastic IP 할당
  - Jenkins Web UI 및 GitHub Webhook 엔드포인트 고정

- **Service-EC2**
  - Elastic IP 할당
  - 외부 사용자 웹 서비스 접근 주소 고정
  - Jenkins SSH 기반 자동 배포 안정성 확보

---

#### Elastic IP 적용 결과

<img width="1231" height="232" alt="image" src="https://github.com/user-attachments/assets/e369302f-78dd-4ec9-b1b0-45689c3c5e18" />

- 인스턴스 중지 및 재시작 이후에도 동일한 IP 주소 유지
- GitHub Webhook 및 Jenkins Pipeline 정상 동작 유지
- CI/CD 파이프라인의 안정성 및 운영 환경 유사성 향상

---

### 2026-01-02 — CI Quality Gate 실패 시나리오 검증 (의도적 FAIL)

Jenkinsfile의 CI(Test – Quality Gate) 단계에는 다음과 같은 테스트 로직이 정의되어 있다.

```bash
stage('Test (Quality Gate)') {
    steps {
        sh '''
        echo "Running CI Test..."
        
        # 최소 품질 게이트 예시
        # index.html 파일이 존재하지 않으면 실패
        test -f index.html

        echo "Test Passed"
        '''
    }
}
```
해당 테스트는 Jenkins 워크스페이스 내에 index.html 파일이 존재하는지를 검사하는
CI 품질 게이트(Quality Gate) 역할의 테스트 로직이다.

본 실습에서는 CI 실패 상황을 의도적으로 발생시키기 위해
기존 ``index.html`` 파일의 이름을 다음과 같이 변경하였다.

```
index.html -> index_fail.html
```
이로 인해 ``test -f index.html`` 조건이 충족되지 않도록 구성하였다.

---

**Jenkins Pipeline 실행 결과**

<img width="543" height="289" alt="image" src="https://github.com/user-attachments/assets/425b5dbc-5121-4a55-8243-bda075c7339f" />
<img width="627" height="696" alt="image" src="https://github.com/user-attachments/assets/a8ebb548-721b-455b-b06b-61778ded1e61" />

- CI(Test – Quality Gate) 단계에서 exit code 1 반환
- Jenkins Pipeline 상태: FAILURE
- 이후 단계 자동 중단
  - Docker Build: skipped
  - Docker Push: skipped
  - Deploy to Service EC2: skipped
 
이는 Jenkins Declarative Pipeline의 기본 동작 방식으로,
앞선 Stage에서 실패가 발생할 경우 이후 Stage를 실행하지 않아
잘못된 빌드 및 배포를 방지하는 정상적인 파이프라인 보호 메커니즘이다.

---

### 2026-01-03 - Nginx Reverse Proxy 도입 및 인프라 확장

앞선 단계에서는 CI 품질 게이트를 통해
**검증되지 않은 코드가 운영 환경에 배포되지 않도록 보호하는 구조**를 완성했다.

다음 단계에서는 **실제 운영 환경에 가까운 서비스 아키텍처**로 확장하기 위해 **Nginx**를 도입한다.

---

### Nginx를 도입하려는 이유

현재 구조에서는 Docker 컨테이너가 다음 역할을 동시에 수행하고 있다.
- 외부 사용자 요청 직접 수신
- 웹 서버 역할 수행
- 애플리케이션 실행 환경 제공

즉, **컨테이너가 서비스의 진입 지점**이 되는 구조였다.

이 방식은 단일 컨테이너 환경에서는 단순하지만,
운영 환경 관점에서는 다음과 같은 한계가 존재한다.

- 컨테이너가 80 포트를 직접 점유하여 확장에 제약 발생
- 컨테이너 재시작 시 서비스 중단
- 다중 컨테이너 운영 및 무중단 배포 구조 구성 어려움
- SSL(HTTPS) 처리 및 트래픽 제어 지점 부재

이러한 문제를 해결하기 위해
**애플리케이션 실행 계층과 트래픽 제어 계층을 분리**할 필요가 있으며,
그 역할을 담당하는 구성 요소가 **Nginx**이다.

---

### Nginx란 무엇인가

Nginx는 단순한 웹 서버가 아니라, 실무 환경에서는 주로 **Reverse Proxy(리버스 프록시)**로 사용된다.

이 프로젝트에서 Nginx의 역할은 다음과 같다.

- 외부 사용자 요청을 가장 먼저 수신하는 **고정 진입 지점**
- 요청을 내부 Docker 컨테이너로 전달하는 **중계자**
- 컨테이너 변경, 재배포 여부와 관계없이 동일한 접속 지점 제공
- 향후 다중 컨테이너, 무중단 배포, HTTPS 확장을 위한 기반 구성

즉,
- **Nginx**: 외부 트래픽 제어 및 진입 관문
- **Docker 컨테이너**: 애플리케이션 실행만 담당
이라는 역할 분리를 통해 서비스 구조를 보다 안정적이고 확장 가능한 상태로 변경한다.

---

### Nginx 도입 전·후 구조 비교

**기존 구조 (Nginx 미사용)**
```
User
  -> Service EC2 : 80
    -> Docker Container : 80
```
- 컨테이너가 외부 요청을 직접 처리
- 컨테이너 재시작 시 서비스 중단
- 포트 충돌로 다중 컨테이너 운영 어려움

---

**Nginx 도입 후 구조**
```
User
  -> Service EC2 (Nginx) : 80
    -> Docker Container : 8080
```
- 외부 사용자 요청은 항상 **Nginx가 수신**
- Docker 컨테이너는 내부 포트에서 애플리케이션만 실행
- 컨테이너 교체·재배포 시에도 외부 접속 지점은 유지
- 실서비스 환경과 유사한 아키텍처 구조 확보

---

### CI/CD 관점에서의 변화

Nginx 도입 이후에도 Jenkins 기반 CI/CD 파이프라인의 핵심 흐름은 동일하다.

```
GitHub Push
-> Jenkins Trigger
-> Test (Quality Gate)
-> Docker Build
-> Docker Push
-> Deploy
```

다만 **Deploy 단계의 의미는 다음과 같이 변경된다.**

- 기존:
  - Docker 컨테이너가 80 포트를 직접 점유
  - 컨테이너 재시작 시 서비스 중단 발생

- Nginx 도입 후:
  - Nginx는 항상 실행된 상태로 유지
  - Jenkins는 **애플리케이션 컨테이너만 교체**
  - 외부 사용자는 서비스 중단 없이 동일한 주소로 접근 가능

즉, Nginx는 배포 대상이 아닌  
**고정된 인프라 레이어(Infrastructure Layer)** 로 동작하게 된다.

---

### Nginx 도입의 의미 정리

Nginx를 도입한다는 것은 단순히 웹 서버를 하나 추가하는 것이 아니라,  
서비스 아키텍처를 다음과 같이 분리하는 것을 의미한다.

- 트래픽 제어 및 외부 진입 지점: **Nginx**
- 애플리케이션 실행 환경: **Docker 컨테이너**
- 빌드·테스트·배포 자동화: **Jenkins**

이를 통해 다음과 같은 효과를 얻을 수 있다.

- 컨테이너 재배포 시 서비스 안정성 향상
- 다중 컨테이너 확장 구조 기반 마련
- 무중단 배포(Blue-Green, Rolling) 확장 가능
- HTTPS 및 보안 설정 확장 용이

---

### Summary

- Jenkins 기반 CI/CD 파이프라인에 CI Quality Gate 적용 완료
- 테스트 실패 시 배포를 자동 차단하는 보호 메커니즘 검증
- Docker 컨테이너 직접 노출 구조에서 탈피
- Nginx Reverse Proxy 도입을 통한 실서비스 아키텍처 확장
- 운영 환경과 유사한 CI/CD + 서비스 구조 구성 완료

---

### Next Step

다음 단계에서는 다음 내용을 추가로 실습 및 확장할 예정이다.

- Nginx Reverse Proxy 설정 (`nginx.conf`)
- Docker 컨테이너 내부 포트 기반 서비스 구성
- Jenkins 배포 스크립트 Nginx 구조에 맞게 수정
- 무중단 배포(Blue-Green) 구조 확장
- HTTPS(SSL/TLS) 적용

이를 통해 단순한 자동 배포를 넘어  
**운영 안정성과 확장성을 고려한 CI/CD 아키텍처**로 완성도를 높일 계획이다.

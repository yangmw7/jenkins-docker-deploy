# jenkins-docker-deploy
Jenkins 기반 Docker CI/CD 자동 배포 실습 프로젝트

### 2026-01-01

#### 🏗️ CI/CD Architecture (Manual Build)

<img width="1536" height="1024" alt="architecture" src="https://github.com/user-attachments/assets/8d8cb6c8-c2d1-4f57-a3f7-ef9446c87d47" />

위 아키텍처는 Jenkins를 사용하여 Docker 기반 애플리케이션을
수동으로 빌드 및 배포하는 CI/CD 파이프라인 구조를 나타냅니다.

개발자는 GitHub 저장소에 애플리케이션 소스를 관리하며,
Jenkins EC2 서버에서 직접 빌드를 실행하면
Jenkins가 소스를 체크아웃하고 Docker 이미지를 빌드합니다.

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


### 2026-01-01

#### 🚀 CI/CD Architecture (GitHub Webhook – Auto Deploy)

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
  3. Docker Hub에 이미지 Push (`latest`)험

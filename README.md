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

본 실습을 통해  
Jenkins Pipeline, Docker 기반 배포, GitHub Webhook 트리거 흐름을 엔드 투 엔드로 경험

(Spring WAS) 애플리케이션 서버 구축

이 문서는 `imskw02` 라즈베리파이(`192.168.0.11`)에 Spring Boot 웹 애플리케이션을 Docker 컨테이너로 배포하는 과정을 안내합니다. Jenkins를 이용한 CI/CD 파이프라인 구축도 포함됩니다.

## 1. 사전 준비

- Ubuntu Server가 설치된 라즈베리파이
- 고정 IP 할당
- Docker, Git, Java 11 (OpenJDK) 설치
- Jenkins 서버 설치 (별도 서버 또는 `imskw02`에 설치 가능)

## 2. Jenkins CI/CD 파이프라인 설정

### 2.1. Jenkinsfile 작성

애플리케이션 레포지토리의 루트에 `Jenkinsfile`이 있어야 합니다. 이 파일은 CI/CD 파이프라인의 각 단계를 코드로 정의합니다. (`application/Jenkinsfile` 참고)

### 2.2. Jenkins 설정

1. **새로운 Item 생성**: Jenkins 대시보드 > `New Item` > `Pipeline` 선택
2. **Pipeline 설정**:
    - `Definition`: `Pipeline script from SCM` 선택
    - `SCM`: `Git` 선택
    - `Repository URL`: GitHub 레포지토리 주소 입력 (`https_주소.git`)
    - `Credentials`: GitHub 접근을 위한 사용자 정보 추가
    - `Branch to build`: `main` 또는 `master`
    - `Script Path`: `Jenkinsfile` (기본값)
3. **Webhook 설정**:
    - GitHub 레포지토리 > `Settings` > `Webhooks` > `Add webhook`
    - `Payload URL`: `http://<Jenkins_서버_주소>/github-webhook/`
    - `Content type`: `application/json`
    - `Secret`: (선택) 보안 토큰
    - `Which events...`: `Just the push event.` 선택

이제 GitHub에 코드를 Push하면 Jenkins가 자동으로 빌드, 테스트, 배포를 진행합니다.

## 3. 수동 배포 (초기 설정 및 테스트용)

Jenkins 설정 전, 수동으로 애플리케이션을 배포하는 방법입니다.

### 3.1. 소스 코드 복제

```
# imskw02 서버에서 애플리케이션 코드를 클론합니다.
git clone <GitHub_레포지토리_주소>
cd <레포지토리_이름>/application

```

### 3.2. 애플리케이션 빌드

Gradle Wrapper를 이용하여 프로젝트를 빌드합니다. `build.gradle`에 따라 의존성을 다운로드하고 `.jar` 파일을 생성합니다.

```
# 실행 권한 부여
chmod +x ./gradlew

# 빌드 실행
./gradlew build

```

### 3.3. Docker 이미지 생성

`Dockerfile`을 이용하여 Spring Boot 애플리케이션을 실행할 Docker 이미지를 만듭니다.

```
# docker build -t <이미지이름:태그> <Dockerfile_위치>
# 예: docker build -t everdo-app:latest .
sudo docker build -t <원하는_이미지_이름>:<태그> .

```

### 3.4. Docker 컨테이너 실행

생성한 이미지로 컨테이너를 실행합니다.

```
# docker run -d --name <컨테이너이름> -p <호스트포트>:<컨테이너포트> <이미지이름:태그>
# 예: sudo docker run -d --name everdo_was -p 8080:8080 everdo-app:latest
sudo docker run -d --name <원하는_컨테이너_이름> -p 8080:8080 <위에서_만든_이미지_이름>:<태그>

```

- **해설**:
    - `d`: 백그라운드 실행
    - `-name`: 컨테이너에 이름 부여
    - `p 8080:8080`: 라즈베리파이의 8080 포트를 컨테이너의 8080 포트와 연결합니다. Spring Boot는 기본적으로 8080 포트를 사용합니다.

이제 `imskw01` 서버를 통해 웹 서비스에 접속할 수 있습니다.

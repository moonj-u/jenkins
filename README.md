# Jenkins 

## 1. Jenkins 개요
```
Jenkins는 소프트웨어 빌드, 테스트, 제공 또는 배포와 관련된 모든 종류의 작업을 자동화하는데 사용할 수 있는 독립형 오픈 소스 자동화 서버입니다.

```  
<br/>

## 2. Jenkins 설치

1. Jenkins 공식 웹 사이트에서 적절한 패키지를 선택하여 다운로드합니다.

2. 각 운영 체제에 맞는 설치 방법은 [Jenkins 공식 문서](https://www.jenkins.io/doc/book/installing/)를 참조하여 Jenkins를 설치합니다.

3. 설치 후 웹 브라우저를 열고 `http://localhost:8080`에 접속하여 초기 설정을 완료합니다.

4. 플러그인 설치 및 관리자 계정을 설정합니다.

<br>

| 구분 | 특징 |
|---|---|
| Jenkins Binary 설치 | - 장점 <br> 1. <br> 2. <br> 3.|
| Jenkins Docker 설치 | - 장점 <br> - 단점 <br> |

<br/>

## 3. 공통 Jenkins 작업

- [Job 생성 방법](docs/job-create.md)
- [Plugin 설치 방법](docs/plugin-install.md)
- [Tools 설정 방법](docs/tools.md)
- [Credentials 설정 방법](docs/credentials.md)
- [GitLab Connection 설정 방법](docs/gitlab-connection.md)

<br/>

## 4. 공통 PlugIn Install

- [Git Plugin](https://plugins.jenkins.io/git/)
- [GitLab Plugin](https://plugins.jenkins.io/gitlab-plugin/)
- [Pipeline Plugin](https://github.com/jenkinsci/workflow-aggregator-plugin)

<br/>

## 5. Jenkins PlugIn 방식을 사용한 빌드 및 배포

### 5-1. Node 프로젝트 빌드

```
Vue3 프로젝트 빌드
```

<br/>

### 5-2. Maven 프로젝트 빌드

```
Spring Boot 프로젝트 빌드
```

<br/>

## 6. Jenkins P/L 방식을 사용한 빌드 및 배포

### 6-1. Node 프로젝트 빌드

```
Vue3 프로젝트 빌드
```

#### 필요 PlugIn

1. [NodeJS Plugin](https://plugins.jenkins.io/nodejs/)

#### Jenkins Pipeline 구성

1. 소스 코드 Clone

- 작업할 디렉토리를 설정한 후, GitLab 저장소에서 main 브랜치를 Clone 합니다.

- GitLab에 대한 인증 정보는 미리 설정해둔 Credentials의 자격 증명을 이용하여 접근합니다.

2. 프로젝트 빌드

- 작업할 디렉터리로 이동한 후, 해당 디렉터리에 `vite`를 설치합니다.

- `npm run build` 명령어를 통해 프로젝트를 빌드 합니다.

> **자세한 사항은 [Vue3 Build Pipeline 구성 파일](pipeline/vue-build-pipeline.md)을 참고하세요.**

<br/>

### 6-2. Gradle 프로젝트 빌드

```
Spring Boot 프로젝트 빌드
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/)

#### Jenkins Pipeline 구성

1. 소스 코드 Clone

- Jenkins 파이프라인에서 작업할 디렉토리를 설정한 후, GitLab 저장소에서 소스 코드를 Clone합니다.

- GitLab에 대한 인증 정보는 미리 설정해둔 Credentials의 자격 증명을 이용하여 접근합니다.

2. 프로젝트 빌드

- 작업할 디렉터리로 이동한 후 `gradle build` 명령어를 통해 프로젝트를 빌드 합니다. 

> **자세한 사항은 [Spring Boot Build Pipeline 구성 파일](pipeline/boot-build-pipeline.md)을 참고하세요.**

<br/>

### 6-3. nginx 형태 배포

```
빌드된 Vue3 프로젝트를 Nginx Server에 배포
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 환경 변수 설정

- 빌드 된 파일이 위치한 디렉터리와 Nginx 서버에서 파일을 저장할 디렉터리를 환경 변수로 설정합니다.

2. 파일 전송

- sshagent 블록을 사용하여 Jenkins에서 설정한 SSH키를 이용해 원격 서버에 인증합니다.

- scp 명령어를 사용하여 dist 폴더를 Nginx 서버에서 파일을 저장할 디렉터리로 전송합니다.

> **자세한 사항은 [Vue3 Deploy Pipeline 구성 파일](pipeline/vue-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-4. war 형태 tomcat 배포

```
빌드된 Spring Boot 프로젝트를 war 형태로 Tomcat Server에 배포
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 환경 변수 설정

- 빌드 된 war 파일의 위치와 Tomcat 서버에서 webapps 폴더의 위치를 환경 변수로 설정합니다.

2. 파일 전송

- sshagent 블록을 사용하여 Jenkins에서 설정한 SSH키를 이용해 원격 서버에 인증합니다.

- scp 명령어를 사용하여 war 파일을 Tomcat 서버에 webapps 폴더로 전송합니다.

3. war 파일 실행

- SSH를 통해 Tomcat 서버에 접속하고, Tomcat 서버의 bin 디렉터리로 이동하여 `startup.sh` 스크립트를 실행하여 Tomcat 서버를 시작합니다.

> **자세한 사항은 [Spring Boot Deploy Pipeline 구성 파일](pipeline/boot-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-5. docker 빌드

```
Spring Boot, Vue3 프로젝트 빌드 및 Docker Image 생성
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/) - Spring Boot 프로젝트

2. [NodeJS Plugin](https://plugins.jenkins.io/nodejs/) - Vue3 프로젝트

3. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 프로젝트 빌드

- Spring Boot를 빌드 : [6-2. Gradle 프로젝트 빌드](#6-2-gradle-프로젝트-빌드)와 동일하게 진행됩니다.

- Vue3 프로젝트 빌드 : [6-1. Node 프로젝트 빌드](#6-1-node-프로젝트-빌드)와 동일하게 진행됩니다.

2. Docker Image 생성

- 프로젝트 빌드 후 `build.gradle`에 설정된 jib 플러그인을 `gradle jib` 명령어를 통해 Image를 생성합니다.

- 해당 방법은 Spring Boot와 Vue3 모두 적용됩니다.

> **자세한 사항은 [Docker Build Pipeline 구성 파일](pipeline/docker-build-pipeline.md)을 참고하세요.**

<br/>

### 6-6. docker 배포

```
생성된 Docker Image를 각 Server에 배포

- Vue3 Docker Image : Nginx Server
- Spring Boot Docker Image : Tomcat Server에 배포
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. sshagent 블록을 사용하여 Jenkins에서 설정한 SSH 키를 이용해 원격 서버에 인증 및 접속합니다.

2. Docker Hub에 로그인합니다.

- `docker login` 명령어를 사용하여 Docker Hub에 로그인합니다.

3. Docker Image 다운로드 및 배포

- `docker pull` 명령어를 이용하여 빌드 시 Docker Hub에 업로드된 Docker Image를 원격 서버로 가져옵니다.

- 가져온 Docker Image를 서버에 배포합니다.

> **자세한 사항은 [Docker Deploy Pipeline 구성 파일](pipeline/docker-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-7. k8s(Kubernetes) 배포

```
Kubernetes 환경에서 Docker Image를 다운로드한 후, YAML 파일을 이용하여 배포
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 


<br/>

### 6-8. Git Flow 배포 전략에 따른 Jenkins P/L

```
Git Flow 배포 전략에 따라 Jenkins 파이프라인을 설정하여 자동으로 병합, 빌드, 배포를 수행
```

#### 필요 Plugin

#### Jenkins Pipeline 구성

<br/>

### 6-9. GitLab Flow 배포 전략에 따른 Jenkins P/L

```
GitLab Flow 배포 전략에 따라 Webhook을 이용하여 Jenkins 파이프라인을 설정하고, 자동으로 병합 및 빌드를 수행
```

#### 필요 Plugin

#### Jenkins Pipeline 구성

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

## 4. Jenkins PlugIn 방식을 사용한 빌드 및 배포

### 4-1. Node 프로젝트 빌드

```
Vue3 프로젝트 빌드
```

<br/>

### 4-2. Maven 프로젝트 빌드

```
Spring Boot 프로젝트 빌드
```

<br/>

## 5. Jenkins P/L 방식을 사용한 빌드 및 배포

### 5-1. Node 프로젝트 빌드

```
Jenkins Pipeline을 이용하여 Vue3 프로젝트를 빌드하고 배포하는 방법
```
#### Jenkins Pipeline 구성

1. 소스 코드 Clone
- GitLab 저장소에서 main 브랜치를 Clone 합니다.
- GitLab에 대한 인증 정보는 미리 설정해둔 Credentials의 자격 증명을 이용하여 접근합니다.

2. 프로젝트 빌드
- 작업할 디렉터리로 이동한 후, 해당 디렉터리에 'vite'를 설치합니다.
- 'npm run build' 명령어를 통해 프로젝트를 빌드 합니다.

> **자세한 사항은 [Vue3 Build Pipeline 구성 파일](pipeline/vue-build-pipeline.md)을 참고하세요.**

<br/>

### 5-2. Maven 프로젝트 빌드

```
Spring Boot 프로젝트 빌드
```
<br/>

### 5-3. nginx 형태 배포

```
빌드된 Vue3 프로젝트를 Nginx Server에 배포
```
<br/>

### 5-4. war 형태 tomcat 배포

```
빌드된 Spring Boot 프로젝트를 war 형태로 Tomcat Server에 배포
```
<br/>

### 5-5. docker 빌드

```
Spring Boot 프로젝트를 빌드 후 Docker Image 생성
```
<br/>

### 5-6. docker 배포

```
생성된 Docker Image를 각 Server에 배포

- Vue3 Docker Image : Nginx Server
- Spring Boot Docker Image : Tomcat Server에 배포
```
<br/>

### 5-7. k8s(Kubernetes) 배포

```
Kubernetes 환경에서 Docker Image를 다운로드한 후, YAML 파일을 이용하여 배포
```
<br/>

### 5-8. Git Flow 배포 전략에 따른 Jenkins P/L

```
Git Flow 배포 전략에 따라 Jenkins 파이프라인을 설정하여 자동으로 병합, 빌드, 배포를 수행
```
<br/>

### 5-9. GitLab Flow 배포 전략에 따른 Jenkins P/L

```
GitLab Flow 배포 전략에 따라 Webhook을 이용하여 Jenkins 파이프라인을 설정하고, 자동으로 병합 및 빌드를 수행
```

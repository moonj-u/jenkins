# Jenkins 

## 1. Jenkins 개요

```
Jenkins는 CI/CD Pipeline을 구축하고 관리하는 데 사용되는 독립형 오픈 소스 자동화 서버입니다.
소프트웨어의 빌드, 테스트, 배포 등의 작업을 자동화하여 개발 프로세스를 효율적으로 관리할 수 있습니다.
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
| Jenkins Binary 설치 | - 장점 <br> 1. 성능 최적화 <br> - 시스템 자원에 맞게 Jenkins를 최적화할 수 있습니다. <br> 2. 세부적인 사용자 정의 가능 <br> - Jenkins의 모든 설정과 구성을 직접 관리할 수 있어, 환경에 맞게 조정이 가능합니다. <br> <br> - 단점 <br> 1. 설치 복잡성 <br> - 운영 체제나 환경에 따라 설치 과정이 복잡할 수 있습니다. <br> 2. 일관성 부족 <br> - 개발 및 운영 환경 간의 일관성을 유지하기 어려울 수 있습니다. <br> 3. 업데이트 관리 <br> - Jenkins의 새로운 버전이나 플러그인 업데이트를 수동으로 관리해야 합니다.  |
| Jenkins Docker 설치 | - 장점 <br> 1. 빠른 배포와 실행 <br> - Docker 컨테이너를 이용하여 Jenkins를 신속하게 배포하고 실행할 수 있습니다. <br> 2. 일관성 유지 <br> - 컨테이너 환경으로 인해 개발 및 운영 환경 간의 일관성을 유지할 수 있습니다. <br> 3. 유지보수 용이 <br> - Docker 이미지를 업데이트하여 Jenkins를 쉽게 유지보수할 수 있습니다. <br> <br> - 단점 <br> 1. 컨테이너 관리 <br> - Docker 컨테이너와 관련된 추가적인 관리 및 설정이 필요합니다. <br> 2. 성능 영향 <br> - 컨테이너가 호스트 시스템과 리소스를 공유하므로 성능에 영향을 미칠 수 있습니다. <br> 3. 보안 문제 <br> - Docker 컨테이너와 호스트 시스템 간의 보안 설정을 주의해야 하며, 잘못된 설정으로 보안 문제를 초래할 수 있습니다. |

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

#### 필요 PlugIn

1. [NodeJS Plugin](https://plugins.jenkins.io/nodejs/)

#### Jenkins Job 구성

1. Freestyle Job 생성

- Jenkins 대시보드에서 `새로운 Item`을 클릭하고, `Freestyle project`를 선택하여 새로운 Job을 생성합니다.

2. 소스 코드 관리

- 소스 코드 관리 섹션에서 Git을 선택합니다.

- GitLab 저장소 URL과 인증 정보를 선택합니다.

3. 빌드 환경

- 빌드 환경 섹션에서 `Provide Node & npm bin/ folder to PATH`을 선택합니다.

- Jenkins 관리 >> Tools 에서 설정한 Node.js Version을 선택합니다.

4. Build Steps

- Build Steps 섹션에서 `Execute shell`을 선택합니다.

- 아래의 명령어를 입력하여 의존성 패키지를 설치하고, Vite를 개발 의존성으로 설치한 후, 프로젝트를 빌드합니다.

```
npm install
npm install vite --save-dev
npm run build
```

> **자세한 사항은 [Vue3 Build Freestyle Job 구성 파일](pipeline/vue-build-freestyle.md)을 참고하세요.**

<br/>

### 5-2. Gradle 프로젝트 빌드

```
Spring Boot 프로젝트 빌드
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/)

#### Jenkins Job 구성

1. Freestyle Job 생성

- Jenkins 대시보드에서 `새로운 Item`을 클릭하고, `Freestyle project`를 선택하여 새로운 Job을 생성합니다.

2. 소스 코드 관리

- 소스 코드 관리 섹션에서 Git을 선택합니다.

- GitLab 저장소 URL과 인증 정보를 선택합니다.

3. Build Steps

- Build Steps 섹션에서 `Invoke Gradle`을 선택합니다.

- Jenkins 관리 >> Tools에서 설정한 Gradle Version을 설정합니다.

- Tasks 섹션에서 `clean`, `build` 명령어를 작성합니다.

```
clean build
```

> **자세한 사항은 [Spring Boot Build Freestyle Job 구성 파일](pipeline/boot-build-freestyle.md)을 참고하세요.**

<br/>

## 6. Jenkins P/L 방식을 사용한 빌드 및 배포

### 6-1. Node 프로젝트 빌드

```
원격 저장소에서 소스 코드를 가져와 Vue3 프로젝트를 빌드
```

#### 필요 PlugIn

1. [NodeJS Plugin](https://plugins.jenkins.io/nodejs/)

#### Jenkins Pipeline 구성

1. 소스 코드 Clone

- Jenkins 파이프라인에서 작업할 디렉터리를 설정한 후, GitLab 저장소에서 소스 코드를 Clone 합니다.

- GitLab에 대한 인증 정보는 미리 설정해둔 `Credentials의 자격 증명`을 이용하여 접근합니다.

2. 프로젝트 빌드

- 작업할 디렉터리로 이동한 후, 해당 디렉터리에 `vite`를 설치합니다.

- `npm run build` 명령어를 통해 프로젝트를 빌드 합니다.

> **자세한 사항은 [Vue3 Build Pipeline 구성 파일](pipeline/vue-build-pipeline.md)을 참고하세요.**

<br/>

### 6-2. Gradle 프로젝트 빌드

```
원격 저장소에서 소스 코드를 가져와 Spring Boot 프로젝트를 빌드
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/)

#### Jenkins Pipeline 구성

1. 소스 코드 Clone

- Jenkins 파이프라인에서 작업할 디렉터리를 설정한 후, GitLab 저장소에서 소스 코드를 Clone 합니다.

- GitLab에 대한 인증 정보는 미리 설정해둔 `Credentials의 자격 증명`을 이용하여 접근합니다.

2. 프로젝트 빌드

- 작업할 디렉터리로 이동한 후 `gradle build` 명령어를 통해 프로젝트를 빌드 합니다. 

> **자세한 사항은 [Spring Boot Build Pipeline 구성 파일](pipeline/boot-build-pipeline.md)을 참고하세요.**

<br/>

### 6-3. nginx 형태 배포

```
빌드 된 Vue3 프로젝트를 Nginx Server에 배포
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 환경 변수 설정

- `Jenkins Server의 dist 폴더의 경로`와 `Nginx Server의 dist 폴더를 배포할 경로`를 환경 변수로 설정합니다.

2. 파일 전송

- `sshagent 블록`을 사용하여 Jenkins에서 설정한 SSH 키를 이용해 원격 서버에 인증합니다.

- `scp` 명령어를 사용하여 dist 폴더를 Nginx 서버에서 파일을 저장할 디렉터리로 전송합니다.

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

- `Jenkins Server의 war 파일의 경로`와 `Tomcat Server의 war 파일을 배포할 경로`를 환경 변수로 설정합니다.

2. 파일 전송

- `sshagent 블록`을 사용하여 Jenkins에서 설정한 SSH 키를 이용해 원격 서버에 인증합니다.

- `scp 명령어`를 사용하여 war 파일을 Tomcat 서버에 webapps 폴더로 전송합니다.

3. war 파일 실행

- SSH를 통해 Tomcat 서버에 접속하고, Tomcat 서버의 bin 디렉터리로 이동하여 `startup.sh` 스크립트를 실행하여 Tomcat 서버를 시작합니다.

> **자세한 사항은 [Spring Boot Deploy Pipeline 구성 파일](pipeline/boot-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-5. docker 빌드

```
Spring Boot, Vue3 프로젝트 빌드 및 Docker Image 생성
```

#### 필요 Plugin

1. [NodeJS Plugin](https://plugins.jenkins.io/nodejs/) - Vue3 프로젝트

2. [Gradle Plugin](https://plugins.jenkins.io/gradle/) - Spring Boot 프로젝트

3. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 프로젝트 빌드

- Vue3 프로젝트 빌드 : [6-1. Node 프로젝트 빌드](#6-1-node-프로젝트-빌드)와 동일하게 진행됩니다.

- Spring Boot를 빌드 : [6-2. Gradle 프로젝트 빌드](#6-2-gradle-프로젝트-빌드)와 동일하게 진행됩니다.

2. Docker Image 생성

- 프로젝트 빌드 후 `build.gradle`에 설정된 jib 플러그인을 `gradle jib` 명령어를 통해 Image를 생성합니다.

- 해당 Pipeline의 구성은 Spring Boot와 Vue3 모두 적용됩니다.

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

1. `sshagent 블록`을 사용하여 Jenkins에서 설정한 SSH 키를 이용해 원격 서버에 인증 및 접속합니다.

2. Docker Hub에 로그인합니다.

- `docker login` 명령어를 사용하여 Docker Hub에 로그인합니다.

3. Docker Image 다운로드 및 배포

- `docker pull` 명령어를 이용하여 빌드 시 Docker Hub에 업로드된 Docker Image를 원격 서버로 가져옵니다.

- 가져온 Docker Image를 `docker run` 명령어로 docker 컨테이너를 실행시킵니다.

> **자세한 사항은 [Docker Deploy Pipeline 구성 파일](pipeline/docker-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-7. k8s(Kubernetes) 배포

```
Kubernetes 환경에서 Docker Image를 다운로드한 후, YAML 파일을 이용하여 배포

Spring Boot, Vue3의 Jenkins Pipeline 구성이 동일
```

#### 필요 Plugin

1. [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)

#### Jenkins Pipeline 구성

1. 환경 변수 설정

- 배포에 필요한 Kubernetes YAML 파일의 경로와 Docker Hub 인증 정보를 환경 변수로 설정합니다.

2. Docker Image 다운로드

- sshagent 블록을 사용하여 Jenkins에서 설정한 SSH 키를 이용해 원격 서버에 인증 및 접속합니다.

- `docker login` 명령어를 사용해 Docker Hub에 로그인하고, `docker pull` 명령어로 해당 Docker 이미지를 다운로드합니다.

3. 원격 서버에 전송한 Kubernetes YAML 파일을 이용하여 `kubectl apply` 명령어로 Kubernetes 클러스터에 배포를 진행합니다.

> **자세한 사항은 [K8s Deploy Pipeline 구성 파일](pipeline/k8s-deploy-pipeline.md)을 참고하세요.**

<br/>

### 6-8. Git Flow 배포 전략에 따른 Jenkins P/L

```
Git Flow 배포 전략에 따라 Jenkins 파이프라인을 설정하여 자동으로 병합 및 빌드 수행
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/)

2. [Git Parameter Plugin](https://plugins.jenkins.io/git-parameter/)

#### Jenkins Pipeline 구성

1. 매개변수 설정

- Boolean Parameter
    1. master 브랜치 작업을 하기 위한 파라미터로 클릭 시 해당 스테이지가 실행됩니다.

- String Parameter
    1. BRANCH_PATTERN
        - 작업을 시작할 브랜치의 이름을 지정합니다.
    2. MERGE_MESSAGE
        - 브랜치를 병합할 때 사용될 커밋 메시지를 지정합니다.
    3. TAG
        - 태그 할 버전 지정
        - 주로 릴리스 버전을 표시하는 데 사용됩니다.
        - 릴리스 브랜치를 master 브랜치와 병합 시 사용되며, 병합 후 master 브랜치를 빌드하고 Docker Image 생성 시 해당 태그가 사용됩니다.

2. Git Checkout

- BRANCH_PATTERN 파라미터를 사용하여 작업할 브랜치를 선택하고, 해당 브랜치를 체크아웃합니다.

3. 각 브랜치의 작업 설명

- Feature Branch 병합
    - feature 브랜치를 develop 브랜치에 병합합니다.

- Release Branch 병합
    - release 브랜치를 develop 브랜치와 master 브랜치에 병합합니다.

- Hotfixes
    - hotfix 브랜치를 master 브랜치에 병합합니다.

- Master Branch
    - master 브랜치에서 최종적으로 빌드 후 Docker 이미지를 생성합니다.
    - 해당 stage는 MASTER_BRANCH 파라미터가 true로 설정된 경우에만 실행됩니다.

> **자세한 사항은 [Git Flow Pipeline 구성 파일](pipeline/gitflow-pipeline.md)을 참고하세요.**

<br/>

### 6-9. GitLab Flow 배포 전략에 따른 Jenkins P/L

```
GitLab Flow 배포 전략에 따라 Jenkins Pipeline을 구성하고, Webhook을 통해 자동으로 병합 및 빌드를 수행
```

#### 필요 Plugin

1. [Gradle Plugin](https://plugins.jenkins.io/gradle/)

2. [Pipeline Utility Steps Plugin](https://plugins.jenkins.io/pipeline-utility-steps/)

    - [Jenkins 공식 문서](https://www.jenkins.io/doc/pipeline/steps/pipeline-utility-steps/)

#### Jenkins Pipeline 구성

- Merge Request Comment Job

    1. Webhook 설정

        - GitLab에서 MR(Merge Request) 요청 시 Webhook을 통해 Jenkins가 트리거됩니다.
        
        - Jenkins 구성에서 `Opened Merge Request Events`를 체크합니다.

    2. 환경 변수 설정

        - GitLab PlugIn에 정의된 변수를 활용해서 Source 브랜치와 Target 브랜치, 그리고 Milesotone의 Iid를 환경 변수로 설정합니다.

        - GitLab API 호출에 필요한 private access token을 환경 변수로 설정합니다.

    3. 병합 및 단위 테스트
    
        - GitLab에서 MR 요청 시 Webhook에 의해 트리거된 Jenkins 파이프라인에서 Source 브랜치와 Target 브랜치를 병합 후 단위 테스트를 진행합니다.

    4. MR(Merge Request)에 Comment 추가
    
        - 테스트 성공 시
        
            - Merge Request에 `unit test success` Comment를 전달합니다.

        - 테스트 실패 시

            - Merge Request에 `unit test failure` Comment를 전달합니다.

> **자세한 사항은 [Merge Request Comment Pipeline 구성 파일](pipeline/mr-comment-pipeline.md)을 참고하세요.**

<br/>

- Pre-Production Merge Request Merge Job

    1. Webhook 설정

        - Webhook을 통해 GitLab에서 MR(Merge Request) 승인 시 Jenkins 파이프라인이 트리거 됩니다.

        - Jenkins 구성에서 `Accepted Merge Request Events`를 체크합니다.

    2. 환경 변수 설정

        - GitLab PlugIn에 정의된 변수를 활용해서 Source 브랜치와 Target 브랜치, 그리고 Milesotone의 Iid를 환경 변수로 설정합니다.

        - GitLab API 호출에 필요한 private access token을 환경 변수로 설정합니다.

    3. Milesotone Title 확인 및 브랜치 체크아웃

        - GitLab에서 code reviewer가 승인 시 Webhook을 통해 전달된 정보로 Milesotone Title을 변수로 설정합니다.

        - Milesotone Title 확인 후 정보가 있을 경우 Target 브랜치에 체크아웃 합니다.

        - pre-production 브랜치로 체크아웃 후 Target 브랜치와 Milesotone의 Title을 이용하여 태그를 생성합니다.

    4. 업로드 및 Revert

        - 테스트 성공 시 
        
            - pre-production 브랜치와 태그를 원격 저장소에 업로드합니다.

        - 테스트 실패 시
        
            - Target 브랜치로 체크아웃하여 revert를 진행 후 원격 저장소에 업로드합니다.

> **자세한 사항은 [GitLab Flow Merge Pipeline 구성 파일](pipeline/gitlabFlow-merge-pipeline.md)을 참고하세요.**

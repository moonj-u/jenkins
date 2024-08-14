# Spring Boot 프로젝트 Kubernetes 배포 Pipeline

## 목차

1. [환경 변수 설정](#1-환경-변수-설정)
2. [YAML 파일 원격 서버 배포 및 Docker Image 다운로드](#2-yaml-파일-원격-서버-배포-및-docker-image-다운로드)
3. [YAML 파일 적용](#3-yaml-파일-적용)
4. [예시) 최종 Pipeline](#예시-최종-pipeline)

## Pipeline 설명

#### 1. 환경 변수 설정

- 배포에 필요한 Kubernetes YAML 파일의 경로와 Docker Hub 인증 정보를 환경 변수로 설정합니다.

<br/>

#### 2. YAML 파일 원격 서버 배포 및 Docker Image 다운로드

- sshagent

    - SSH Agent Plugin을 설치한 후, Pipeline에서 sshagent 블록을 사용하여 SSH 자격 증명을 제공합니다.

    - sshagent 블록은 지정된 SSH 자격 증명을 사용하여 블록 내에서 실행되는 모든 셸 명령에 SSH 키를 제공할 수 있습니다.

    ```groovy
    stages {
        stage('') {
            steps {
                sshagent(credentials: ['자격 증명 ID']) {
                    //
                }
            }
        }
    }
    ```
<br/>

- YAML 파일 원격 서버로 배포

    - `scp` 명령어를 이용하여 YAML 파일을 Kubernetes Server에 전송합니다.

    ```
    scp [옵션] [폴더명] [원격지_id]@[원격지_ip]:[받는 위치]
    ```
<br/>

- Docker Hub 로그인 및 Docker Image 다운로드

    - `ssh [원격지_id]@[원격지_ip]` 명령어로 원격 서버로 접속합니다.

    - `echo $DOCKERHUB_CREDENTIALS_PSW`

        - 환경 변수에 저장된 Docker Hub 비밀번호를 출력합니다.

        - 비밀번호는 실제로 log에 출력되지 않고, `docker login` 명령어의 표준 입력으로 전달됩니다.

    - `docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin`

        - `-u` 옵션에서 Docker Hub 사용자 이름을 지정합니다.

        - `--password-stdin` 옵션을 사용하여 비밀번호를 표준 입력(stdin)에서 읽어오는데, 해당 Pipeline에서는 `echo` 명령어로 전달된 비밀번호를 사용합니다.

    - &&
        
        - Docker Hub 로그인 후의 작업을 위해 해당 연산자를 사용합니다.

    - `docker pull docker.io/[Docker Hub ID]/[Docker Image]:[TAG]`

        - Docker의 기본 퍼블릭 레지스트리인 `docker.io`에서 다운로드하려는 Docker 이미지의 이름과 TAG를 지정하여 `docker pull` 명령어로 이미지를 다운로드합니다.

>**참고** <br/>
>[Docker Login 공식 문서](https://docs.docker.com/reference/cli/docker/login) <br/>
>[Jenkins 자격 증명 처리 공식 문서](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#usernames-and-passwords)

<br/>

#### 3. YAML 파일 적용

- `cd` 명령어로 YAML 파일이 다운로드 된 경로로 이동합니다.

- `apply -f [파일명.yaml]` 명령어를 사용하여 YAML 파일을 실행시킵니다.

<br/>

## 예시) 최종 Pipeline

### Spring Boot

```groovy
pipeline {
    agent any
    
    environment {
        DEPLOYMENT_YAML = '/var/jenkins_home/workspace/mj-docker-boot-build/backend/k8s/boot-deployment.yaml'   // Jenkins Server의 YAML 파일 경로
        SERVICE_YAML = '/var/jenkins_home/workspace/mj-docker-boot-build/backend/k8s/boot-service.yaml'         // Jenkins Server의 YAML 파일 경로
        MANIFAST_DIR = '/home/nginx/manifest'   // 원격 서버의 YAML 파일을 배포할 폴더 경로
        DOCKERHUB_CREDENTIALS = credentials('자격 증명 ID')
    }

    stages {
        stage('docker imgae pull') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh """
                            scp ${DEPLOYMENT_YAML} ${SERVICE_YAML} [원격지_id]@[원격지_ip]:${MANIFAST_DIR}
                            ssh [원격지_id]@[원격지_ip] "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/[Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                            "
                        """
                    }
                }
            }
        }
        
        stage('k8s yaml file apply') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh """
                            ssh [원격지_id]@[원격지_ip] "
                                cd ${MANIFAST_DIR}
                                kubectl apply -f boot-deployment.yaml
                                kubectl apply -f boot-service.yaml
                            "
                        """
                    }
                }
            }
        }
    }
}
```

<br/>

### Vue3

```groovy
pipeline {
    agent any
    
    environment {
        DEPLOYMENT_YAML = '/var/jenkins_home/workspace/mj-docker-vue-build/frontend/k8s/vue-deployment.yaml'    // Jenkins Server의 YAML 파일 경로
        SERVICE_YAML = '/var/jenkins_home/workspace/mj-docker-vue-build/frontend/k8s/vue-service.yaml'          // Jenkins Server의 YAML 파일 경로
        MANIFAST_DIR = '/home/nginx/manifest'   // 원격 서버의 YAML 파일을 배포할 폴더 경로
        DOCKERHUB_CREDENTIALS = credentials('자격 증명 ID')
    }

    stages {
        stage('docker imgae pull') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh '''
                            scp ${DEPLOYMENT_YAML} ${SERVICE_YAML} [원격지_id]@[원격지_ip]:${MANIFAST_DIR} 
                            ssh [원격지_id]@[원격지_ip] "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/[Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                            "
                        '''
                    }
                }
            }
        }
        
        stage('k8s yaml file apply') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh """
                            ssh [원격지_id]@[원격지_ip] "
                                cd ${MANIFAST_DIR}
                                kubectl apply -f vue-deployment.yaml
                                kubectl apply -f vue-service.yaml
                            "
                        """
                    }
                }
            }
        }
    }
}
```

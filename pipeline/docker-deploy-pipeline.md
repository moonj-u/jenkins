# Spring Boot, Vue3 프로젝트 Docker Image 배포 Pipeline

## Pipeline 설명

1. environment

- 미리 설정 해둔 `username with password` 자격 유형의 자격 증명을 환경 변수로 설정합니다.

```groovy
environment {
    DOCKERHUB_CREDENTIALS = credentials('자격 증명 ID')
}
```

<br/>

2. sshagent

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

3. Docker Hub 로그인

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

4. Docker Image 실행

- `ssh` 명령어를 통해 원격 서버에 접속합니다.

- 이미 실행 중인 Docker 컨테이너가 있을 수 있으므로, `docker stop` 명령어로 해당 컨테이너를 중지 시킵니다.

- 중지된 Docker 컨테이너를 `docker rm` 명령어로 삭제합니다.

- `docker run` 명령어를 통해 이미지를 실행시킵니다.
    - `-d` 옵션을 사용하여 백그라운드에서 실행시킵니다. 
    - `-p` 옵션을 통해 호스트와 컨테이너의 포트 번호를 매핑하여 지정합니다.
    - `--name` 옵션을 사용하여 새로 생성되는 Docker 컨테이너의 이름을 지정합니다.

<br/>

## 예시) 최종 Pipeline

### Spring Boot

```groovy
pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('자격 증명 ID')
    }
    
    stages {
        stage('boot deploy') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                    sh """
                        ssh tomcat@[원격지_ip] "
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin && 
                        docker pull docker.io/[Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                        "
                    """
                    }
                }
            }
        }
        
        stage('boot run') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh '''
                            ssh tomcat@[원격지_ip] "
                            docker stop [컨테이너 이름]
                            docker rm [컨테이너 이름]
                            docker run -d -p [호스트 포트번호]:[컨테이너 포트번호] --name [컨테이너 이름] [Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                            "
                        '''
                    }
                }
            }
        }
    }
}
```

### Vue3

```groovy
pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('자격 증명 ID')
    }

    stages {
        stage('vue deploy') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh """
                            ssh nginx@원격지_ip "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/[Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                            "
                        """
                    }
                }
            }
        }
        
        stage('vue run') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh '''
                            ssh nginx@원격지_ip "
                            docker stop [컨테이너 이름]
                            docker rm [컨테이너 이름]
                            docker run -d -p [호스트 포트번호]:[컨테이너 포트번호] --name [컨테이너 이름] [Docker Hub 사용자 이름]/[Docker Image]:[TAG]
                            "
                        '''
                    }
                }
            }
        }
    }
}
```

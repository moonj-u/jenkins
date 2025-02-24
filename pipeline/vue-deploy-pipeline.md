# Vue3 프로젝트 nginx 형태 배포 Pipeline

## 목차

1. [환경 변수 설정](#1-환경-변수-설정)
2. [dist 폴더 원격 서버로 전송](#2-dist-폴더-원격-서버로-전송)
3. [예시) 최종 Pipeline](#예시-최종-pipeline)

## Pipeline 설명

#### 1. 환경 변수 설정

- `Jenkins Server의 dist 폴더의 경로`와 `Nginx Server의 dist 폴더를 배포할 경로`를 environment 블록 내에 환경 변수로 설정합니다.

```groovy
environment {
    //
}
```

<br>

#### 2. dist 폴더 원격 서버로 전송

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

<br>

- scp 명령어

    - scp 명령어를 이용하여 dist 폴더를 Tomcat Server에 전송합니다.

    - `-r` 옵션을 사용하여 폴더를 배포합니다.

    ```
    scp [옵션] [폴더명] [원격지_id]@[원격지_ip]:[받는 위치]
    ```

<br>

## 예시) 최종 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        FRONT_DIR = "/var/jenkins_home/workspace/mj-vuebuild/frontend/dist" // Jenkins Server의 dist 폴더 경로
        TARGET_DIR = '/home/nginx/project'  // Nginx Server의 dist 폴더를 배포할 경로
    }

    stages {
        stage('vue deploy') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh 'scp -r ${FRONT_DIR} nginx@[원격지_ip]:${TARGET_DIR}'
                    }
                }
            }
        }
    }
}
```

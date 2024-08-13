# Spring Boot 프로젝트를 War 형태로 Tomcat Server 배포 Pipeline

## Pipeline 설명

1. environment

- 환경 변수를 설정합니다.

```groovy
environment {
    //
}
```

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

3. scp 명령어

- scp 명령어를 이용하여 war 파일을 Tomcat Server에 전송합니다.

- 해당 Pipeline에서는 옵션을 사용하지 않습니다.

```groovy
scp [옵션] [파일명] [원격지_id]@[원격지_ip]:[받는 위치]
```

4. ssh 명령어

- `ssh` 명령어를 사용하여 원격 서버에 접속 후 스크립트를 실행합니다.

- 서버에 접속 후 스크립트를 실행하기 위해 `ssh [원격지_id]@[원격지_ip]` 뒤에 `""(큰따옴표)`를 사용합니다.

```groovy
ssh [원격지_id]@[원격지_ip] "명령어1, 명령어2, 명령어3, ..."
```

<br/>

## 예시) 최종 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        BACK_WAR_FILE = '/var/jenkins_home/workspace/...'   // Jenkins Server의 war 파일 경로
        TAGET_DIR = '/home/tomcat/apache-tomcat-10.1.24/webapps'    // Tomcat Server의 war 파일을 배포할 경로
    }

    stages {
        stage('boot deploy') {
            steps {
                sshagent(credentials: ['자격 증명 ID']) {
                    sh 'scp ${BACK_WAR_FILE} tomcat@[원격지_ip]:${TAGET_DIR}'
                }
            }
        }
        
        stage('run') {
            steps {
                script {
                    sshagent(credentials: ['자격 증명 ID']) {
                        sh '''
                            ssh tomcat@[원격지_ip] "
                            export JAVA_HOME=/home/tomcat/amazon-corretto-17.0.11.9.1-linux-x64
                            export PATH=$JAVA_HOME/bin:$PATH
                            cd /home/tomcat/apache-tomcat-10.1.24/bin
                            sh startup.sh
                            "
                        '''
                    }
                    
                }
            }
        }
    }
}
```

>**참고** : 파일이나 폴더의 경로는 실제 환경에 따라 다를 수 있습니다.

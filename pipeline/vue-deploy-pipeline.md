# Vue3 프로젝트 nginx 형태 배포 Pipeline

```groovy

pipeline {
    agent any
    
    environment {
        FRONT_DIR = "/var/jenkins_home/workspace/mj-vuebuild/frontend/dist"
        TARGET_DIR = '/home/nginx/project'
    }

    stages {
        stage('vue deploy') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh 'scp -r ${FRONT_DIR} nginx@192.168.0.252:${TARGET_DIR}'
                    }
                }
            }
        }
    }
}

```

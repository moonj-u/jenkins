# Spring Boot 프로젝트를 War 형태로 Tomcat Server 배포

```groovy
pipeline {
    agent any
    
    environment {
        BACK_WAR_FILE = '/var/jenkins_home/workspace/mj-bootbuild/backend/build/libs/boot-app.war'
        TAGET_DIR = '/home/tomcat/apache-tomcat-10.1.24/webapps'
    }

    stages {
        stage('boot deploy') {
            steps {
                sshagent(credentials: ['tomcat-ssh-key']) {
                    sh 'scp ${BACK_WAR_FILE} tomcat@192.168.0.252:${TAGET_DIR}'
                }
            }
        }
        
        stage('run') {
            steps {
                script {
                    sshagent(credentials: ['tomcat-ssh-key']) {
                        sh '''
                            ssh tomcat@192.168.0.252 "
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

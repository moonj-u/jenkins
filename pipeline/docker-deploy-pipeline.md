# Spring Boot 프로젝트 Docker Image 배포 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerHub')
    }
    
    stages {
        stage('boot deploy') {
            steps {
                script {
                    sshagent(credentials: ['tomcat-ssh-key']) {
                    sh """
                        ssh tomcat@192.168.0.252 "
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin && 
                        docker pull docker.io/leemoonju/boot-app:latest
                        "
                    """
                    }
                }
            }
        }
        
        stage('boot run') {
            steps {
                script {
                    sshagent(credentials: ['tomcat-ssh-key']) {
                        sh '''
                            ssh tomcat@192.168.0.252 "
                            docker stop boot-app
                            docker rm boot-app
                            docker run -d -p 8080:8080 --name boot-app leemoonju/boot-app:latest
                            "
                        '''
                    }
                }
            }
        }
    }
}
```

<br/>

# Vue3 프로젝트 Docker Image 배포 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerHub')
    }

    stages {
        stage('vue deploy') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh """
                            ssh nginx@192.168.0.252 "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/leemoonju/vue-app:latest
                            "
                        """
                    }
                }
            }
        }
        
        stage('vue run') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh '''
                            ssh nginx@192.168.0.252 "
                            docker stop vue-app
                            docker rm vue-app
                            docker run -d -p 8090:8090 --name vue-app leemoonju/vue-app:latest
                            "
                        '''
                    }
                }
            }
        }
    }
}
```

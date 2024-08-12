# Spring Boot 프로젝트 Kubernetes 배포 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DEPLOYMENT_YAML = '/var/jenkins_home/workspace/mj-docker-boot-build/backend/k8s/boot-deployment.yaml'
        SERVICE_YAML = '/var/jenkins_home/workspace/mj-docker-boot-build/backend/k8s/boot-service.yaml'
        MANIFAST_DIR = '/home/nginx/manifest'
        DOCKERHUB_CREDENTIALS = credentials('dockerHub')
    }

    stages {
        stage('docker imgae pull') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh """
                            scp ${DEPLOYMENT_YAML} ${SERVICE_YAML} nginx@192.168.0.252:${MANIFAST_DIR}
                            ssh nginx@192.168.0.252 "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/leemoonju/boot-app:$tagName
                            "
                        """
                    }
                }
            }
        }
        
        stage('k8s yaml file apply') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh """
                            ssh nginx@192.168.0.252 "
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

# Vue3 프로젝트 Kubernetes 배포 Pipeline

```groovy
pipeline {
    agent any
    
    environment {
        DEPLOYMENT_YAML = '/var/jenkins_home/workspace/mj-docker-vue-build/frontend/k8s/vue-deployment.yaml'
        SERVICE_YAML = '/var/jenkins_home/workspace/mj-docker-vue-build/frontend/k8s/vue-service.yaml'
        MANIFAST_DIR = '/home/nginx/manifest'
        DOCKERHUB_CREDENTIALS = credentials('dockerHub')
    }

    stages {
        stage('docker imgae pull') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh '''
                            scp ${DEPLOYMENT_YAML} ${SERVICE_YAML} nginx@192.168.0.252:${MANIFAST_DIR} 
                            ssh nginx@192.168.0.252 "
                            echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin &&
                            docker pull docker.io/leemoonju/vue-app:$tagName
                            "
                        '''
                    }
                }
            }
        }
        
        stage('k8s yaml file apply') {
            steps {
                script {
                    sshagent(credentials: ['nginx-ssh-key']) {
                        sh """
                            ssh nginx@192.168.0.252 "
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

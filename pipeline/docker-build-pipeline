# Spring Boot 프로젝트 빌드 및 Docker Image 생성 Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('mj-boot-gradle')
    }
    
    stages {
        stage('git clone') {
            steps {
                dir('backend') {
                        git branch: 'master', 
                        changelog: false, 
                        credentialsId: 'mj.lee',
                        poll: false,
                        url: 'https://gitlab.magnum.i234.me/study/backendtomcat.git'
                }
            }
        }
        
        stage('boot build') {
            steps {
                dir('backend') {
                    sh '''
                        gradle build
                        gradle jib
                    '''
                }
            }
        }
    }
}
```

# Vue3 프로젝트 빌드 및 Docker Image 생성 Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('mj-boot-gradle')
        nodejs('mj-vue-nodeJS')
    }
    
    stages {
        stage('vue git clone') {
            steps {
                dir('frontend') {
                    git branch: 'main',
                    changelog: false,
                    credentialsId: 'mj.lee',
                    poll: false,
                    url: 'https://gitlab.magnum.i234.me/study/frontend-vue-tomcat.git' 
                }
            }
        }
        
        stage('vue build') {
            steps {
                dir('frontend') {
                    sh '''
                        npm install vite --save-dev
                        npm run build
                        gradle jib
                    '''
                }
            }
        }
    }
}
```

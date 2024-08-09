# Jenkins Vue3 프로젝트 빌드 파이프라인

```groovy
pipeline {
    agent any
    
    tools {
        gradle('mj-boot-gradle')
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
                        gradle npmDevBuild
                    '''
                }
            }
        }
    }
}
```

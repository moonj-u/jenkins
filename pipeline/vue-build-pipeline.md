# vue-build-pipeline

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

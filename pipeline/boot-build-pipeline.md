## Spring Boot Build Pipeline

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

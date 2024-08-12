# Spring Boot Build Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('Tools Name')
    }
    
    stages {
        stage('git clone') {
            steps {
                dir('working-directory') {
                        git branch: 'master',
                        changelog: false,
                        credentialsId: '자격 증명 ID',
                        poll: false,
                        url: '원격 저장소 URL'
                }
            }
        }
        
        stage('boot build') {
            steps {
                dir('working-directory') {
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

## Pipeline 설명

1. tools

- Jenkins가 사용할 도구의 이름을 지정합니다.

2. dir()

- 작업할 디렉터리를 지정합니다.

- 지정된 디렉터리에 Git Clone 작업이 진행됩니다.

3. git branch

- 설정한 브랜치를 클론합니다.

4. changelog: false

- 변경 로그를 가져오지 않도록 설정합니다.

5. credentialsId: 'credentials-id'

- 저장소 접근을 위한 자격 증명 ID를 설정합니다.

6. poll: false

- Git 저장소의 변경 사항을 주기적으로 확인하지 않도록 설정합니다.

- 해당 사항은 선택 사항이며, 해당 Pipeline에서는 주기적인 변경 사항을 확인하여 빌드할 필요가 없기 때문에 false로 설정합니다.

7. url: 'https://github.com/your-repo.git'

- 클론할 원격 저장소의 URL을 지정합니다.

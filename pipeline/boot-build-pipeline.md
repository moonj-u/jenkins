# Spring Boot Build Pipeline

## 목차

1. [Tools](#1-tools)
2. [git clone](#2-git-clone)
3. [예시) 최종 Pipeline](#예시-최종-pipeline)

## Pipeline 설명

#### 1. Tools

- Jenkins가 사용할 도구의 이름을 지정합니다.

- Tools Name은 Jenkins 관리 > Tools에서 설정한 도구의 이름입니다.

```groovy
    tools {
        gradle('Tools Name')
    }
```

<br>

#### 2. git clone

- dir()

    - 작업할 디렉터리를 지정합니다.

    - 지정된 디렉터리에 Git Clone 작업이 진행됩니다.

    ```groovy
        stage('git clone') {
            steps {
                dir('working-directory')
            }
        }
    ```

<br>

- git branch

    - 설정한 브랜치를 Clone 합니다.

    ```groovy
    git branch: '브랜치명'
    ```

<br>

- changelog(선택 사항)

    - 변경 로그를 가져오지 않도록 설정합니다.

    ```groovy
    changelog: false
    ```

<br>

- credentialsId: 'credentials-id'

    - 저장소 접근을 위한 자격 증명 ID를 설정합니다.

    ```groovy
    credentialsId: '자격 증명 ID'
    ```

<br>

- poll(선택 사항)

    - Git 저장소의 변경 사항을 주기적으로 확인하지 않도록 설정합니다.

    - 해당 사항은 선택 사항이며, 해당 Pipeline에서는 주기적인 변경 사항을 확인하여 빌드 할 필요가 없기 때문에 false로 설정합니다.

    ```groovy
    poll: false
    ```

<br>

- url

    - 클론할 원격 저장소의 URL을 지정합니다.

    ```groovy
    url: '원격 저장소 URL'
    ```

<br/>

## 예시) 최종 Pipeline

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
                        git branch: '브랜치명',
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
                    '''
                }
            }
        }
    }
}
```

# Vue3 build pipeline

## 목차

1. [Tools](#1-tools)
2. [Git Clone](#2-git-clone)
3. [Vue3 Pipeline에서 vite 설치](#3-vue3-pipeline에서-vite-설치)
4. [예시) 최종 Pipeline](#예시-최종-pipeline)

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

#### 2. Git Clone

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

    ```
    git branch: '브랜치명'
    ```

<br>

- changelog(선택 사항)

    - 변경 로그를 가져오지 않도록 설정합니다.

    ```
    changelog: false
    ```

<br>

- credentialsId: 'credentials-id'

    - 저장소 접근을 위한 자격 증명 ID를 설정합니다.

    ```
    credentialsId: '자격 증명 ID'
    ```

<br>

- poll(선택 사항)

    - Git 저장소의 변경 사항을 주기적으로 확인하지 않도록 설정합니다.

    - 해당 사항은 선택 사항이며, 해당 Pipeline에서는 주기적인 변경 사항을 확인하여 빌드 할 필요가 없기 때문에 false로 설정합니다.

    ```
    poll: false
    ```

<br>

- url

    - 클론할 원격 저장소의 URL을 지정합니다.

    ```
    url: '원격 저장소 URL'
    ```

<br>

#### 3. Vue3 Pipeline에서 vite 설치

- Vite는 실제 서비스에서는 필요하지 않기 때문에 `--save-dev` 옵션을 사용하여 Vite를 package.json 파일의 devDependencies 섹션에 추가합니다.

- 해당 옵션을 사용하면 Vite가 개발 환경에서만 의존성으로 포함됩니다.

```
npm install vite --save-dev
```

<br/>

## 예시) 최종 Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        nodejs('Tools Name')
    }
    
    stages {
        stage('vue git clone') {
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
        
        stage('vue build') {
            steps {
                dir('working-directory') {
                    sh '''
                        npm install vite --save-dev
                        npm run build
                    '''
                }
            }
        }
    }
}
```

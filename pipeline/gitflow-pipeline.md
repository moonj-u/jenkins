# Git Flow 배포 전략에 따른 Jenkins Pipeline

## 목차

1. [Tools](#1-tools)
2. [Git Checkout](#2-git-checkout)
3. [when 지시어를 활용한 브랜치별 Stage 실행](#3-when-지시어를-활용한-브랜치별-stage-실행)
4. [stage 공통 작업](#4-stage-공통-작업)
5. [develop branch를 release branch에 병합](#5-develop-branch를-release-branch에-병합)
6. [develop branch를 master branch에 병합](#6-develop-branch를-master-branch에-병합)
7. [master branch 빌드 및 Docker Image 생성](#7-master-branch-빌드-및-docker-image-생성)
8. [예시) 최종 Pipeline](#예시-최종-pipeline)

## Pipeline 설명

#### 1. Tools

- Jenkins가 사용할 도구의 이름을 지정합니다.

- Tools Name은 Jenkins 관리 > Tools에서 설정한 도구의 이름입니다.

```groovy
    tools {
        gradle('Tools Name')
    }
```

<br/>

#### 2. Git Checkout

- deleteDir()

    - 이 메서드는 현재 디렉터리와 그 내용을 재귀적으로 삭제하는 메서드입니다.

    - Git 리포지터리에서 소스 코드를 체크아웃 및 병합 과정에서 이전 빌드의 파일로 인한 충돌 방지를 위해 현재 디렉터리를 삭제하고, 작업 공간을 최신 상태로 유지하기 위해서 해당 메서드를 사용합니다.

- checkout scmGit

    - Jenkins Pipeline에서 Git 리포지터리에서 소스 코드를 체크아웃 하기 위한 스텝입니다.

    ```
    checkout scmGit(...)
    ```

- branches

    - `String Parameter`를 설정하여 Pipeline 빌드 시 매개변수 값을 전달할 수 있도록 합니다.

    ```
    branches : [[name : "origin/${params.PARAMETER_NAME"]]
    ```

- userRemoteConfigs

    - Git 리포지터리와 연결하기 위한 설정입니다.

    - 원격 저장소의 URL과 Jenkins에서 설정한 자격 증명 정보를 사용하여 소스 코드를 다운로드할 수 있습니다.

    ```
    userRemoteConfigs : [[...]]
    ```

<br/>

#### 3. when 지시어를 활용한 브랜치별 Stage 실행

- `when` 지시어를 사용하면 파이프라인에서 주어진 조건에 따라 스테이지를 실행해야 하는지 여부를 결정할 수 있습니다.

- `steps`는 `when` 블록 밑에 위치하여 그 조건이 참일 경우에만 `steps` 블록이 실행됩니다.

```groovy
stage ('') {
    when {
        expression {
            // 조건
        }
    }
    steps {
        // 실행할 스텝
    }
}
```

<br/>

#### 4. stage 공통 작업

- 매개변수 설정

    - Boolean Parameter

        - master 브랜치 작업을 하기 위한 파라미터로 클릭 시 해당 스테이지가 실행됩니다.

    - String Parameter

        - BRANCH_PATTERN
        
            - 작업을 시작할 브랜치의 이름을 지정합니다.

        - MERGE_MESSAGE
        
            - 브랜치를 병합할 때 사용될 커밋 메시지를 지정합니다.

        - TAG

            - 태그 할 버전 지정

            - 주로 릴리스 버전을 표시하는 데 사용됩니다.

            - 릴리스 브랜치를 master 브랜치와 병합 시 사용되며, 병합 후 master 브랜치를 빌드하고 Docker Image 생성 시 해당 태그가 사용됩니다.

- `when` 블록 조건의 변수 설정

    - `returnStdout: true`

        - 명령어의 출력 결과를 표준 출력으로 반환합니다.

        - 명령어의 결과를 변수에 저장할 수 있습니다.

    - `script: 'git rev-parse --abbrev-ref HEAD'`

        - 현재 Git 브랜치의 이름을 반환하는 명령어입니다.

        - `--abbrev-ref` 옵션은 Git에서 참조를 짧은 이름으로 반환합니다. 즉, 전체 객체 이름(커밋, 해시 등) 대신 짧은 형태로 이름을 출력합니다.

        - `HEAD`는 Git에서 현재 작업 중인 브랜치나 커밋을 가리키는 참조입니다.

```groovy
def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()

return git_branch = '[브랜치명]'
```

- 브랜치 병합

    - `git fetch --prune` 명령어를 사용하여 원격 저장소의 최신 변경 사항을 가져오고, `--prune` 옵션을 통해 더 이상 존재하지 않는 원격 브랜치를 로컬 저장소에서 제거합니다.

    - git 로컬 사용자 정보 설정

        - 전역 사용자와 로컬 사용자가 다를 경우 적용시킵니다.

        ```
        git config user.name "[사용자명]"
        git config user.emali "[이메일]"

        ```

    - 병합해야 할 브랜치로 체크아웃 합니다.

    - `git merge --no-ff [Target 브랜치] -m "[Commit 메세지]"`

        - `--no-ff` 옵션을 사용하여 병합 시 fast-forward 병합을 방지하고, 항상 새로운 커밋을 생성하여 병합을 기록합니다.

- 단위 테스트

    - `gradle test` 명령어를 사용하여 단위 테스트를 진행합니다.

    - 테스트 성공 시

        - 병합된 브랜치를 원격 저장소에 업로드합니다.

        - 원격 저장소에서 불필요한 브랜치를 삭제합니다.

    - 테스트 실패 시

        - `git diff` 명령어를 사용하여 Log를 출력합니다.

        - `git merge --abort` 명령어를 사용하여 병합 이전의 상태로 되돌아갑니다.

<br/>

#### 5. develop branch를 release branch에 병합

- `when`블록 조건 

    - `return git_branch ==~ /^release-\d+\.\d+\.\d+$/`

        - 정규 표현식을 사용하여 브랜치 이름이 `release-`로 시작하고 뒤에 세 개의 숫자가 점('.')으로 구분되어 있는 경우에만 steps가 실행됩니다.

- 테스트 성공 시 `develop branch`로 체크아웃하여 `master branch`에 병합하는 stage가 실행될 수 있도록 합니다.

#### 6. develop branch를 master branch에 병합

- `master branch`에 병합 후 `tag`를 생성하는 작업을 진행

    - `git tag -f -a [Milestone Title] -m "[Commit 메세지]"`

        - `-f` 옵션은 이미 존재하는 태그를 덮어쓰는 옵션으로, 기존 태그를 업데이트하거나 수정해야할 때 유용합니다.

        - `-a` 옵션은 태그에 주석을 달아 태그에 대한 설명을 추가하는 목적으로 사용합니다.

<br/>

#### 7. master branch 빌드 및 Docker Image 생성

- 현재 Git 브랜치가 `master`이고, `Boolean Parameter`로 설정해놓은 `MASTER_BRANCH`가 `true`인 경우에 실행됩니다.

- `git fetch --prune --tags` 명령어를 사용하여 원격 저장소의 최신 태그를 가져오고, 해당 태그로 체크아웃 합니다.

- 프로젝트 빌드 후 `build.gradle`에 설정된 jib 플러그인을 `gradle jib` 명령어를 통해 Docker Image를 생성합니다.

<br/>

## 예시) 최종 Pipeline
```groovy
pipeline {
    agent any
    
    tools {
        gradle('Tools Name')
    }
    
    stages {
        stage('git checkout') {
            steps {
                deleteDir()
                checkout scmGit(
                    branches : [[name: "origin/${params.BRANCH_PATTERN}"]],
                    userRemoteConfigs: [[
                        credentialsId: '자격 증명 ID',
                        url: '원격 저장소 URL'
                    ]]
                )
            }
        }
        // feature branch를 develop branch에 병합하는 스테이지
        stage ('feature branch merge to develop') {
            when {
                expression {
                    def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                    
                    return git_branch == 'feature'
                }
            }
            stages {
                stage('merge') {
                    steps {
                        script {
                            sh """
                                git fetch --prune
                                git config user.name "이문주"
                                git config user.email mj.lee@magnumbridge.co.kr
                                git checkout develop
                                git merge --no-ff feature -m "${params.MERGE_MESSAGE}"
                            """
                        }
                    }
                }
                // 병합 후 단위 테스트
                // 현재는 Test Code가 없기 때문에 항상 success
                stage('test') {
                    steps {
                        sh 'gradle test'
                    }
                }
            }
            post {
                // 테스트 성공 시 feature branch 삭제 및 develop branch push
                success {
                    script {
                        echo 'test success'
                        sh """
                            git push origin develop
                            git push origin --delete feature                 
                        """
                    }
                }
                // 테스트 실패 시 rollback
                failure {
                    script {
                        echo 'log를 확인해주세요.'
                        echo 'merge rollback'
                        sh """
                            git diff
                            git merge --abort
                            git status
                        """
                    }
                }
            }
        }
        
        // develop branchrelease branch에서 develop branch에 병합하는 스테이지
        stage ('release branch merge to develop') {
            when {
                expression {
                    def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()

                    return git_branch ==~ /^release-\d+\.\d+\.\d+$/
                }
            }
            stages {
                stage('merge') {
                    steps {
                        script {
                            sh """
                                git fetch --prune
                                git config user.name "이문주"
                                git config user.email mj.lee@magnumbridge.co.kr
                                git checkout develop
                                git merge --no-ff ${params.BRANCH_PATTERN} -m "${params.MERGE_MESSAGE}"
                            """
                        }
                    }
                }
                // 병합 후 단위 테스트
                stage('test') {
                    steps {
                        sh 'gradle test'
                    }
                }
            }
            post {
                // 병합 및 테스트 성공 시 develop branch push
                // master branch에도 동일한 작업이 필요하기 때문에 release branch는 삭제하지 않는다.
                // master branch에서도 동일한 작업을 위해 release branch로 다시 checkout을 해준다.
                success {
                    echo 'test success'
                    sh """
                        git push origin develop
                        git checkout ${params.BRANCH_PATTERN}
                    """
                }
                // 병합 및 테스트 실패 시 rollback
                failure {
                    script {
                        echo 'log를 확인해주세요.'
                        echo 'merge rollback'
                        sh """
                            git diff
                            git merge --abort
                            git status
                        """
                    }
                }
            }
        }
        // release branch에서 master branch에 병합하는 스테이지
        stage('release branch merge to master') {
            when {
                expression {
                    def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()

                    return git_branch ==~ /^release-\d+\.\d+\.\d+$/
                }
            }
            stages {
                stage('merge') {
                    steps {
                        script {
                            sh """
                                git config user.name "이문주"
                                git config user.email mj.lee@magnumbridge.co.kr
                                git checkout master
                                git merge --no-ff ${params.BRANCH_PATTERN} -m "${params.MERGE_MESSAGE}"
                                git tag -f -a ${params.TAG} -m "commit tag version ${params.TAG}"
                            """
                        }
                    }
                }
                // 병합 후 단위 테스트
                stage('test') {
                    steps {
                        sh 'gradle test'
                    }
                }
            }
            post {
                // 병합 및 테스트 성공 시 업로드
                success {
                    sh """
                        git push origin master
                        git push origin ${params.TAG}
                        git push origin --delete ${params.BRANCH_PATTERN}
                    """
                }
                // 병합 및 테스트 실패 시 rollback
                failure {
                    script {
                        echo 'log를 확인해주세요.'
                        echo 'merge rollback'
                        sh """
                            git diff
                            git merge --abort
                            git status
                        """
                    }
                }
            }
        }
        // 
        stage('hotfixes') {
            when {
                expression {
                    def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                    
                    return git_branch ==~ /^hotfix-\d+\.\d+\.\d+$/
                }
            }
            stages {
                stage('merge') {
                    steps {
                        script {
                            sh """
                                git config user.name "이문주"
                                git config user.email mj.lee@magnumbridge.co.kr
                                git fetch --prune
                                git checkout master
                                git merge --no-ff ${params.BRANCH_PATTERN} -m "${params.MERGE_MESSAGE}"
                                git tag -f -a ${params.TAG} -m "commit tag version ${params.TAG}"
                            """
                        }
                        
                    }
                }
                // 병합 후 단위 테스트
                stage('test') {
                    steps {
                        sh 'gradle test'
                    }
                }
            }
            post {
                success {
                    sh """
                        git push origin master
                        git push origin ${params.TAG}
                    """
                }
                // 병합 및 테스트 실패 시 rollback
                failure {
                    script {
                        echo 'log를 확인해주세요.'
                        echo 'merge rollback'
                        sh """
                            git diff
                            git merge --abort
                            git status
                        """
                    }                    
                }
            }
        }
        
        // master branch에서 최종 release를 build하고 Docker image를 생성하는 스테이지
        stage('master branch') {
            when {
                expression {
                    def build_start = "${params.MASTER_BRANCH}"
                    def git_branch = sh(returnStdout: true, script: 'git rev-parse --abbrev-ref HEAD').trim()
                    
                    return build_start == 'true' && git_branch == 'master'
                }
            }
            steps {
                script {
                    sh """
                        git config user.name "이문주"
                        git config user.email mj.lee@magnumbridge.co.kr
                        git fetch --prune --tags
                        git checkout tags/${params.TAG}
                        gradle build
                        gradle jib
                    """
                }
            }
        }
    }
}
```

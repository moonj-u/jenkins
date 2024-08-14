# GitLab Merge Request Comment 작성 pipeline

## 목차

1. [Tools](#1-tools)
2. [환경 변수 설정](#2-환경-변수-설정)
3. [Git Checkout](#3-git-checkout)
4. [브랜치 병합](#4-브랜치-병합)
5. [단위 테스트](#5-단위-테스트)
6. [예시) 최종 Pipeline](#6-예시-최종-pipeline)

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

#### 2. 환경 변수 설정

- GitLab PlugIn에 정의된 변수를 활용해서 Source 브랜치와 Target 브랜치, 그리고 Milesotone의 Iid를 환경 변수로 설정합니다.

- GitLab API 호출에 필요한 private access token을 환경 변수로 설정합니다.

- private access token은 GitLab >> Settings >> Access Tokens에서 생성할 수 있습니다.

```groovy
environment {
    //
}
```

<br/>

#### 3. Git Checkout

- deleteDir()

    - 이 메서드는 현재 디렉터리와 그 내용을 재귀적으로 삭제하는 메서드입니다.

    - Git 리포지터리에서 소스 코드를 체크아웃 및 병합 과정에서 이전 빌드의 파일로 인한 충돌 방지를 위해 현재 디렉터리를 삭제하고, 작업 공간을 최신 상태로 유지하기 위해서 해당 메서드를 사용합니다.

<br/>

- checkout scmGit

    - jenkins Pipeline에서 Git 리포지터리에서 소스 코드를 체크아웃 하기 위한 스탭입니다.

    ```
    checkout scmGit(...)
    ```

<br/>

- branches

    - checkout 할 브랜치를 지정합니다.

    ```
    branches : [[name : "브랜치명"]]
    ```

<br/>

- userRemoteConfigs

    - Git 리포지터리와 연결하기 위한 설정입니다.

    - 원격 저장소의 URL과 Jenkins에서 설정한 자격 증명 정보를 사용하여 소스 코드를 다운로드할 수 있습니다.

    ```
    userRemoteConfigs : [[...]]
    ```

<br/>

#### 4. 브랜치 병합

- `git fetch origin`

    - 원격 저장소의 변경 사항을 로컬 저장소로 가져옵니다.

- `git checkout [Target 브랜치]`

    - 병합할 Target 브랜치로 체크아웃합니다.

- `git merge [Source 브랜치]`

    - Source 브랜치를 Target 브랜치로 병합합니다.

    - 해당 Pipeline에서 단위 테스트를 위한 병합이며, 테스트 진행 후 Merge Request의 Comment만 기록하는 목적이기 때문에 `--no-commit` 옵션을 사용하여 병합 작업이 완료된 후 자동으로 커밋 되지 않게 합니다.

<br/>

#### 5. 단위 테스트

- `gradle test` 명령어로 단위 테스트를 진행하고, `post 블록`을 사용하여 테스트 결과에 따라 후속 작업을 진행합니다.

- 테스트 성공 시 `unit test success`라는 Comment를 `curl` 명령어를 통해 전달합니다.

- 테스트 실패 시 `unit test failure`라는 Comment를 `curl` 명령어를 통해 전달합니다.

- `curl` 명령어를 사용하여 GitLab API에 POST 요청을 보냅니다.

    - `--request POST`

        - POST 요청을 지정합니다.
    
    - `--header`

        - 요청 헤더를 추가합니다.

        - 인증 정보나 데이터 형식을 지정할 때 사용됩니다.

        - `PRIVATE-TOKEN` : GitLab API 인증을 위한 헤더입니다.

        - `Content-Type` : 요청 본문의 Content Type을 지정합니다.

    - `--url`

        - API 요청을 보낼 URL입니다.

    - `--data`

        - 요청 본문에 포함할 데이터를 지정합니다.

>**참고** <br/>
>[GitLab REST API 공식 문서](https://docs.gitlab.com/ee/api/rest/) <br/>
>[GitLab Merge Requests API 공식 문서](https://docs.gitlab.com/ee/api/merge_requests.html)

<br/>

## 예시) 최종 Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('Tools Name')
    }
    
    environment {
        SOURCE_BRANCH = "${env.gitlabSourceBranch}"
        TARGET_BRANCH = "${env.gitlabTargetBranch}"
        MR_IiD = "${env.gitlabMergeRequestIid}"
        GITLAB_PRIVATE_TOKEN = 'glpat-TAyaq4sMpPzPkMxE7if9'
    }
    
    stages {
        stage('checkout') {
            steps {
                deleteDir()
                checkout scmGit(
                    branches : [[name : "origin/${SOURCE_BRANCH}"]],
                    userRemoteConfigs : [[
                        credentialsId : '자격 증명 ID',
                        url : '원격 저장소 URL'
                    ]]
                )
            }
        }
        
        stage('merge') {
            steps {
                script {
                    sh """
                        git fetch origin
                        git checkout origin/${TARGET_BRANCH}
                        git merge origin/${SOURCE_BRANCH} --no-ff --no-commit
                    """
                }
            }
        }
        
        stage('test') {
            steps {
                sh 'gradle test'
            }
        }
    }
    
    post {
        success {
            script {
                echo 'unit test success'
                sh """
                    curl --request POST \
                         --header "PRIVATE-TOKEN: ${GITLAB_PRIVATE_TOKEN}" \
                         --header "Content-Type: application/json" \
                         --url "[project_url]/api/v4/projects/[project_id]/merge_requests/${MR_IiD}/notes" \
                         --data '{"body" : "unit test success"}'
                """
            }
        }
        
        failure {
            echo 'unit test failure'
            
            sh """
                curl --request POST \
                     --header "PRIVATE-TOKEN: ${GITLAB_PRIVATE_TOKEN}" \
                     --header "Content-Type: application/json" \
                     --url "[project_url]/api/v4/projects/[project_id]/merge_requests/${MR_IiD}/notes" \
                     --data '{"body" : "unit test failure"}'
            """
        }
    }
}
```

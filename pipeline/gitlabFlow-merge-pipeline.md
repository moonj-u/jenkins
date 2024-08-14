# GitLab Flow 배포 전략에 따른 pre-production merge pipeline

## 목차

1. [tools](#1-tools)
2. [환경 변수 설정](#2-환경-변수-설정)
3. [Milestone 정보 가져오기](#3-milestone-정보-가져오기)
4. [Target 브랜치를 pre-production 브랜치에 병합 및 테스트](#4-target-브랜치를-pre-production-브랜치에-병합-및-테스트)
5. [예시) 최종 Pipeline](#예시-최종-pipeline)

## Pipeline 설명

#### 1. tools

- Jenkins가 사용할 도구의 이름을 지정합니다.

- Tools Name은 Jenkins 관리 > Tools에서 설정한 도구의 이름입니다.

```groovy
    tools {
        gradle('Tools Name')
    }
```

<br/>

#### 2. 환경 변수 설정

- GitLab PlugIn에 정의된 변수를 활용해서 Source 브랜치와 Target 브랜치, 그리고 Milestone의 Iid를 환경 변수로 설정합니다.

- GitLab API 호출에 필요한 private access token을 환경 변수로 설정합니다.

- private access token은 GitLab >> Settings >> Access Tokens에서 생성할 수 있습니다.

<br/>

#### 3. Milestone 정보 가져오기

- Script Pipeline

    - `def` 변수와 복잡한 작업을 처리하기 위해 선언적 파이프라인이 아닌 스크립트 파이프라인을 사용합니다.

    - `script 블록`을 활용하여 Groovy 코드를 작성합니다.

    ```groovy
    stages {
        stage() {
            steps {
                script {
                    //
                }
            }
        }
    }
    ```

    <br/>

- Merge Request API 호출 및 변수 설정

    - `def [변수명] = sh(script: ..., returnStdout: true).trim()`

        - `returnStdout: true`

            - 명령어의 출력 결과를 표준 출력으로 반환합니다.

            - 명령어의 결과를 변수에 저장할 수 있습니다.

        - `trim`

            - 출력된 결과의 앞뒤 공백을 제거합니다.

        - `curl` 명령어를 사용하여 Merge Request API에 GET 요청을 보냅니다.

            - `--header`

                - 요청 헤더를 추가합니다.

                - 인증 정보나 데이터 형식을 지정할 때 사용합니다.

                - `PRIVATE-TOKEN` : GitLab API에 접근하기 위한 개인 액세스 토큰을 포함하는 헤더입니다.

            - `url`

                - API 요청을 보낼 URL입니다.

>**참고** <br/>
>[GitLab REST API 공식 문서](https://docs.gitlab.com/ee/api/rest/) <br/>
>[GitLab Merge Requests API 공식 문서](https://docs.gitlab.com/ee/api/merge_requests.html)

<br/>

- readJSON 스텝 및 변수 설정

    - `def jsonProps = readJSON text: props`

        - readJSON 스텝을 사용하여 JSON 형식의 문자열을 Groovy 객체로 변환합니다.

        - `text: props`
            
            - JSON 형식의 문자열이 저장된 변수를 의미합니다. 
            
            - sh 명령어를 통해 가져온 API 응답을 JSON 객체로 파싱 하여 변수에 저장합니다.

    - `def milestoneTitle = jsonProps.milestone.title`

        - 파싱 된 JSON 객체(jsonProps)에서 milestone 필드의 title 값을 추출하여 milestoneTitle 변수에 저장합니다.

        - GitLab Merge Request의 마일스톤 제목을 나타냅니다.

    - `def milestoneId = jsonProps.milestone.id`

        - JSON 객체에서 milestone 필드의 id 값을 추출하여 milestoneId 변수에 저장합니다.

        - GitLab Merge Request의 마일스톤 ID를 나타냅니다.

    - `env.`

        - 추출한 Milestone의 Title과 ID 값을 Jenkins 환경 변수로 설정합니다.

<br/>

#### 4. Target 브랜치를 pre-producion 브랜치에 병합 및 테스트

- deleteDir()

    - 이 메서드는 현재 디렉터리와 그 내용을 재귀적으로 삭제하는 메서드입니다.

    - Git 리포지터리에서 소스 코드를 체크아웃 및 병합 과정에서 이전 빌드의 파일로 인한 충돌 방지를 위해 현재 디렉터리를 삭제하고, 작업 공간을 최신 상태로 유지하기 위해서 해당 메서드를 사용합니다.

- checkout scmGit

    - Jenkins Pipeline에서 Git 리포지터리에서 소스 코드를 체크아웃 하기 위한 스텝입니다.

    ```
    checkout scmGit(...)
    ```

- branches

    - checkout 할 브랜치를 지정합니다.

    ```
    branches : [[name : "브랜치명"]]
    ```

- userRemoteConfigs

    - Git 리포지터리와 연결하기 위한 설정입니다.

    - 원격 저장소의 URL과 Jenkins에서 설정한 자격 증명 정보를 사용하여 소스 코드를 다운로드할 수 있습니다.

    ```
    userRemoteConfigs : [[...]]
    ```

- 병합
    - git 로컬 사용자 정보 설정

        - 전역 사용자와 로컬 사용자가 다를 경우 적용시킵니다.

        ```groovy
        git config user.name "[사용자명]"
        git config user.emali "[이메일]"
        ```

    - 병합을 위해 pre-producion 브랜치로 체크아웃 합니다.

    - `git merge --no-ff [Target 브랜치] -m "[Commit 메세지]"`
    
        - `--no-ff` 옵션을 사용하여 병합 시 fast-forward 병합을 방지하고, 항상 새로운 커밋을 생성하여 병합을 기록합니다.
    
    - `git tag -f -a [Milestone Title] -m "[Commit 메세지]"`
    
        - `-f` 옵션은 이미 존재하는 태그를 덮어쓰는 옵션으로, 기존 태그를 업데이트하거나 수정해야할 때 유용합니다.

        - `a` 옵션은 태그에 주석을 달아 태그에 대한 설명을 추가하는 목적으로 사용합니다.


>**참고** <br/>
>[Git merge 공식 문서](https://git-scm.com/docs/git-merge) <br/>
>[Git tag 공식 문서](https://git-scm.com/docs/git-tag)

<br/>

- 단위 테스트

    - `gradle test` 명령어를 통해 단위 테스트를 진행 합니다.

    - 테스트 성공 시 
    
        - 병합된 pre-producion 브랜치와 태그를 원격 저장소에 업로드 합니다.

    - 테스트 실패 시
    
        - `git log origin/${TARGET_BRANCH} --pretty="%H" -n 1` 명령어를 사용하여 최신 커밋의 해시 값을 가져와 변수로 설정합니다.
        
        - 변수로 설정한 해시 값을 사용하여 해당 커밋을 `git revert -m 1 [변수명]` 명령어를 사용하여 변경 사항을 되돌립니다.

        - 되돌린 변경 사항을 원격 저장소에 업로드합니다.

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
        stage('Get Milestone Title') {
            steps {
                script {
                    def props = sh(script: """
                        curl --header "PRIVATE-TOKEN: ${GITLAB_PRIVATE_TOKEN}" \
                             --url "https://gitlab.magnum.i234.me/api/v4/projects/93/merge_requests/${MR_IiD}" 
                             """, returnStdout: true).trim()
                    
                    def jsonProps = readJSON text: props
                    def milestoneTitle = jsonProps.milestone.title
                    def milestoneId = jsonProps.milestone.id
                    env.MILESTONE_TITLE = milestoneTitle
                    env.MILESTON_ID = milestoneId
                    
                }
            }
        }
        
        stage('Target Branch Job') {
            when {
                expression {
                    env.MILESTONE_TITLE != null
                }
            }
            stages {
                stage('Target Branch Checkout') {
                    steps {
                        deleteDir()
                        checkout scmGit(
                            branches : [[name : "origin/${SOURCE_BRANCH}"]],
                            userRemoteConfigs : [[
                                credentialsId : 'mj.lee',
                                url : 'https://gitlab.magnum.i234.me/study/backendtomcat.git'
                            ]]
                        )
                    }                   
                }
                
                stage('Target Branch Merge to pre-producion branch') {
                    steps {
                        script {
                            sh """
                                git config user.name "이문주"
                                git config user.email "mj.lee@magnumbridge.co.kr"
                                git branch
                                git checkout pre-production
                                git merge --no-ff ${TARGET_BRANCH} -m "${env.MILESTONE_TITLE} version merge to pre-production"
                                git tag -f -a ${env.MILESTONE_TITLE} -m "commit tag version ${env.MILESTONE_TITLE}"
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
                        sh """
                            git push origin pre-production
                            git push origin ${env.MILESTONE_TITLE}
                        """
                    }
                }
                failure {
                    script {
                        echo 'unit test fail merge rollback'
                    
                        def commitHash = sh(returnStdout: true, script: 'git log origin/${TARGET_BRANCH} --pretty="%H" -n 1').trim()
                        
                        sh """
                            git config user.name "이문주"
                            git config user.email mj.lee@magnumbridge.co.kr
                            git checkout ${TARGET_BRANCH}
                            git revert -m 1 ${commitHash}
                            git push origin ${TARGET_BRANCH}
                        """
                    }
                }
            }
        }
    }
}
```

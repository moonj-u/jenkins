# GitLab Flow 배포 전략에 따른 pre-production merge pipeline

## Pipeline 설명

1. tools

- Jenkins가 사용할 도구의 이름을 지정합니다.

- Tools Name은 Jenkins 관리 > Tools에서 설정한 도구의 이름입니다.

```groovy
    tools {
        gradle('Tools Name')
    }
```

2. 환경 변수 설정

- GitLab PlugIn에 정의된 변수를 활용해서 Source 브랜치와 Target 브랜치, 그리고 Milestone의 Iid를 환경 변수로 설정합니다.

- GitLab API 호출에 필요한 private access token을 환경 변수로 설정합니다.

- private access token은 GitLab >> Settings >> Access Tokens에서 생성할 수 있습니다.

3. Milestone Title 가져오기

- `def` 변수를 사용하기 위해 script 블록 내부에서 Groovy 코드를 작성합니다.

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

3. deleteDir()

- 이 메서드는 현재 디렉터리와 그 내용을 재귀적으로 삭제하는 메서드입니다.

- Git 리포지터리에서 소스 코드를 체크아웃 및 병합 과정에서 이전 빌드의 파일로 인한 충돌 방지를 위해 현재 디렉터리를 삭제하고, 작업 공간을 최신 상태로 유지하기 위해서 해당 메서드를 사용합니다.



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
                                git config user.email mj.lee@magnumbridge.co.kr
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

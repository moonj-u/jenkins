# Git Flow 배포 전략에 따른 Jenkins Pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('mj-boot-gradle')
    }
    
    stages {
        stage('git checkout') {
            steps {
                deleteDir()
                checkout([$class: 'GitSCM',
                    branches: [[name: "origin/${params.BRANCH_PATTERN}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'LocalBranch']],
                    submoduleCfg: [],
                    userRemoteConfigs: [[
                        credentialsId: 'mj.lee',
                        url: 'https://gitlab.magnum.i234.me/study/backendtomcat.git']]])
                        
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

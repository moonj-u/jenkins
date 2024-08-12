# GitLab Flow 배포 전략에 따른 pre-production merge pipeline

```groovy
pipeline {
    agent any
    
    tools {
        gradle('mj-boot-gradle')
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
                        checkout ([$class : 'GitSCM',
                        branches : [[name : "origin/${TARGET_BRANCH}"]],
                        extensions: [[$class : 'LocalBranch']],
                        userRemoteConfigs : [[
                            credentialsId : 'mj.lee',
                            url : 'https://gitlab.magnum.i234.me/study/backendtomcat.git'
                            ]]
                        ])
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

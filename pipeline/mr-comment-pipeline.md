# GitLab Merge Request Comment 작성 pipeline

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
        stage('checkout') {
            steps {
                deleteDir()
                checkout ([$class : 'GitSCM',
                branches : [[name : "origin/${SOURCE_BRANCH}"]],
                extensions: [[$class : 'LocalBranch']],
                userRemoteConfigs : [[
                    credentialsId : 'mj.lee',
                    url : 'https://gitlab.magnum.i234.me/study/backendtomcat.git'
                    ]]
                ])                
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
                         --url "https://gitlab.magnum.i234.me/api/v4/projects/93/merge_requests/${MR_IiD}/notes" \
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
                     --url "https://gitlab.magnum.i234.me/api/v4/projects/93/merge_requests/${MR_IiD}/notes" \
                     --data '{"body" : "unit test failure"}'
            """
        }
    }
}
```

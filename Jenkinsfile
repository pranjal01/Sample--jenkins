pipeline {
    agent any

    environment{
        registry = "pranjal01/demo-helm-chart"
        registryCredential = "local-docker"
        dockerImage = ''
        DockerfilePath = "Dockerfile"
              
    }
    stages {
        stage('sementic-release') {
            steps{
                withCredentials(
                        [   usernamePassword(credentialsId: 'local-docker', 
                            passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME'), 
                            usernameColonPassword(credentialsId: 'gitlab_cred', variable: 'GIT_CREDENTIALS'),
                        ]
                    ) {
                       sh "sed -i -e \"s#IMAGE_NAME#${registry}#g\" .releaserc"
                       sh "sed -i -e \"s#Dockerfile#${DockerfilePath}#g\" .releaserc"
                       sh "sed -i -e \"s#COMMIT_ID#${GIT_COMMIT}#g\" basic-structure/src/app/routes/version.json"
                       sh "sed -i -e \"s#BUILD_DATE#`date`#g\" basic-structure/src/app/routes/version.json"
                       sh "sed -i -e \"s#BRANCH#${GIT_BRANCH}#g\" basic-structure/src/app/routes/version.json"
                       sh " cat .releaserc"
                       sh "/usr/bin/semantic-release --no-ci --debug"
                       sh "export `cat version`"
                }
            }
        }
        stage('Remove Unused Dockerimage') {
            steps{
                sh 'export `cat version` && docker rmi $registry:${SEM_VERSION}'
           }
        }
        stage('Secrets Copy') {
            steps{
                    withCredentials([file(credentialsId: 'LocalKubernetes', variable: 'LocalKubernetes')]) {
                        sh "cp \$LocalKubernetes config"
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps{
                sh 'export `cat version` && helm --kubeconfig=config upgrade -i gold-code-backend-$BRANCH_NAME -n $namespace basic-structure/gold-code-backend-$BRANCH_NAME --set image.tag=${SEM_VERSION}'
            }
        }

        stage('Workspace Cleanup') {
            steps{
                sh 'echo $VERSION'
                cleanWs()
            }
        }
    }
}

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
               
                       sh "sed -i -e \"s#IMAGE_NAME#${registry}#g\" .releaserc"
                       sh "sed -i -e \"s#Dockerfile#${DockerfilePath}#g\" .releaserc"
                       sh " cat .releaserc"
                
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
                sh 'export `cat version` && helm --kubeconfig=config upgrade -i sample-helm -$BRANCH_NAME -n $namespace Sample-jenkins/sample-helm-$BRANCH_NAME --set image.tag=${SEM_VERSION}'
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

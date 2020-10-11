pipeline {
    agent any

    stages {
        stage ('Clone') {
            steps {
                git branch: 'master', url: "https://github.com/talitz/spring-petclinic-ci-cd-k8s-example.git"
            }
        }

        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "talyi-artifactory",
                    url: "https://talyi.jfrog.io/artifactory",
                    credentialsId: "admin.jfrog"
                )
            }
        }

        stage ('Build docker image') {
            steps {
                script {
                    docker.build("talyi-docker.jfrog.io/" + 'pet-clinic:latest')
                }
            }
        }

        stage ('Push image to Artifactory') {
            steps {
                rtDockerPush(
                    serverId: "talyi-artifactory",
                    image: "talyi-docker.jfrog.io/" + 'pet-clinic:latest',
                    targetRepo: 'docker',
                    properties: 'project-name=jfrog-blog-post;status=stable'
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "talyi-artifactory"
                )
            }
        }

        stage('Install helm') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'talyi-artifactory', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                              //sh "curl -u ${env.USERNAME}:${env.PASSWORD} -O talyi.jfrog.io/generic-local/helm"
                              //sh "chmod 777 ./helm"

                              sh """
                              curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3"
                              chmod 700 get_helm.sh
                              helm version
                              ./get_helm.sh
                              """
                }
            }
        }

        stage('Configure helm & add Artifactory repo') {
            steps {
                 sh './get_helm init'
                 withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'talyi-artifactory', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                   sh "./get_helm repo add artifactory talyi.jfrog.io/helm ${env.USERNAME} ${env.PASSWORD}"
                   sh "./get_helm repo update"
                   sh "./get_helm helm version"
                 }
            }
        }

        stage('Deploy chart pulling from Artifactory') {
            steps {
                sh "./get_helm upgrade spring-petclinic-ci-cd-k8s-example.tgz --install artifactory/spring-petclinic-ci-cd-k8s-example.tgz"
            }
        }
    }
}

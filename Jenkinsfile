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
                  sh "curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3"
                  sh "chmod 700 get_helm.sh && helm version"
            }
        }

        stage('Configure helm & add Artifactory repo') {
            steps {
                 withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'talyi-artifactory', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                   sh "helm init"
                   sh "helm repo add artifactory talyi.jfrog.io/helm ${env.USERNAME} ${env.PASSWORD}"
                   sh "helm repo update"
                   sh "helm helm version"
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

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
                    docker.build("talyi.jfrog.io/docker" + '/pet-clinic:latest')
                }
            }
        }

        stage ('Push image to Artifactory') {
            steps {
                rtDockerPush(
                    serverId: "talyi-artifactory",
                    image: "talyi.jfrog.io/docker" + '/pet-clinic:latest',
                    targetRepo: 'docker-local',
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
    }
}

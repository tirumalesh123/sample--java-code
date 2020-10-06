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
                    url: SERVER_URL,
                    credentialsId: CREDENTIALS
                )
            }
        }

        stage ('Build docker image') {
            steps {
                script {
                    docker.build(ARTIFACTORY_DOCKER_REGISTRY + '/pet-clinic:latest', 'jenkins-examples/pipeline-examples/resources')
                }
            }
        }

        stage ('Push image to Artifactory') {
            steps {
                rtDockerPush(
                    serverId: "talyi-artifactory",
                    image: ARTIFACTORY_DOCKER_REGISTRY + '/petclinic:latest',
                    targetRepo: 'docker-local',
                    properties: 'project-name=docker1;status=stable'
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

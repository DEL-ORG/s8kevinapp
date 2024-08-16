pipeline {
    agent {label "SERVER03"}

    triggers {
        githubPush()
    }

    environment {
        DOCKER_HUB_USERNAME = "devopseasylearning"
        WEB_APP = "vag-app"
        DOCKER_CREDENTIAL_ID = 's8-test-docker-hub-auth'
    }

    parameters {
        string (name: 'BRANCH_NAME', defaultValue: "main", description: 'Branch name to build')
        string (name: 'App', defaultValue: "new", description: 'Tag for application')
        string (name: 'PORT_ON_DOCKER_HOST', defaultValue: "", description: 'Port on Docker host')

    }

    stage ('Clone Repo'){
        when {
            expression{
                params.BRANCH_NAME == 'main'
            }
        }
        steps{
            script{
                git credentialsId: 'jenkins-ssh-agents-private-key'
                url: 'git@github.com:DEL-ORG/s8kevinapp.git'
                branch: '${params.BRANCH_NAME}'
            }
        }
    }

    stage('Building Vagrant App'){
            when {
                expression {
                    params.BRANCH_NAME == 'main'
                }
            }
            steps {
                script {
                    sh """
                        docker build -t ${env.DOCKER_HUB_USERNAME}/app-01:${BUILD_NUMBER} .
                        docker images
                    """
            }
        }
    }

    stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL_ID}", 
                    usernameVariable: 'DOCKER_USERNAME', 
                    passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Use Docker CLI to login
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL_ID}", 
                    usernameVariable: 'DOCKER_USERNAME', 
                    passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Use Docker CLI to login
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    }
                }
            }
        }

        stage('Push Images to Docker hUb'){
            steps{
                script{
                    sh """
                        docker push ${env.DOCKER_HUB_USERNAME}/app-01:${BUILD_NUMBER}
                        docker push ${env.DOCKER_HUB_USERNAME}/s8-sonar-scanner-app:latest
                    """
                }
            }
        }

        stage('Test Timeout') {
            steps {
                script {
                    sleep 1 // This stage is for testing purposes; adjust as needed
                }
            }
        }

}
def sanity_check() {
    if (params.BRANCH_NAME.isEmpty()) {
        error "The parameter BRANCH_NAME is not set"
    }
}



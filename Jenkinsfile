pipeline {
    agent { label "SERVER03" }

    triggers {
        githubPush()
    }

    environment {
        DOCKER_HUB_USERNAME = "devopseasylearning"
        DOCKER_CREDENTIAL_ID = 's8-test-docker-hub-auth'
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch name to build')
        string(name: 'App', defaultValue: 'new', description: 'Tag for application')
        string(name: 'PORT_ON_DOCKER_HOST', defaultValue: '', description: 'Port on Docker host')
    }

    stages {
        stage('Sanity Check') {
            steps {
                script {
                    sanity_check()
                }
            }
        }

        stage('Clone Repo') {
            when {
                expression { params.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    git credentialsId: 'jenkins-ssh-agents-private-key',
                        url: 'git@github.com:DEL-ORG/s8kevinapp.git',
                        branch: "${params.BRANCH_NAME}"
                }
            }
        }

        stage('Building Docker Images') {
            when {
                expression { params.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    sh """
                        docker build -t ${env.DOCKER_HUB_USERNAME}/app-01:${BUILD_NUMBER} .
                        docker build -t ${env.DOCKER_HUB_USERNAME}/s8-sonar-scanner-app:latest .
                        docker images
                    """
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL_ID}", 
                                                      usernameVariable: 'DOCKER_USERNAME', 
                                                      passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                    }
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
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
}

def sanity_check() {
    if (params.BRANCH_NAME.isEmpty()) {
        error "The parameter BRANCH_NAME is not set"
    }
}

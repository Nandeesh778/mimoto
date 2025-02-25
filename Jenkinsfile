pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/Nandeesh778/mimoto.git'
        GIT_BRANCH = 'test' 
        DOCKER_IMAGE_BASE = 'raparna154/inji-mimoto-service'
        MANIFEST_REPO = 'https://github.com/Aparnadeloitte/Inji-infra-azure.git'
        MANIFEST_BRANCH = 'main'
    }

    stages {
        stage('Get Commit Hash') {
            steps {
                script {
                    env.COMMIT_HASH = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                    env.DOCKER_IMAGE = "${DOCKER_IMAGE_BASE}:${env.COMMIT_HASH}-${env.BUILD_NUMBER}"
                    echo "Docker Image Tag: ${env.DOCKER_IMAGE}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${env.DOCKER_IMAGE} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhubpat', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${env.DOCKER_IMAGE}
                        docker logout
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! updated image: ${env.DOCKER_IMAGE} in manifest repo."
        }
        failure {
            echo "Failed! Last attempted Docker Image Tag: ${env.DOCKER_IMAGE}"
        }
    }
}
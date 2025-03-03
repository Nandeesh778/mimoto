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
        stage('Clean Workspace') {
            steps {
                sh 'rm -rf mimoto || true'
            }
        }

        stage('Clone Repository') {
            steps {
                sh """
                git clone ${GIT_REPO}
                cd mimoto
                git checkout ${GIT_BRANCH}
                git pull
                """
            }
        }

        stage('Get Commit Hash') {
            steps {
                script {
                    env.COMMIT_HASH = sh(
                        script: "cd mimoto && git rev-parse --short HEAD",
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
                    dir('mimoto') {
                        sh """ mvn clean package -DskipTests """
                        sh """
                        docker build -t ${env.DOCKER_IMAGE} .
                        """
                    }
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

        stage('Update Manifest Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'githubpat', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                        rm -rf mimoto-infra
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/your-org/mimoto-infra.git
                        cd mimoto-infra
                        git checkout ${MANIFEST_BRANCH}

                        echo "Before update:"
                        cat mimoto/values.yaml || true

                        yq eval '.image.repository = "${DOCKER_IMAGE_BASE}" |
                                .image.tag = "'"${env.COMMIT_HASH}-${env.BUILD_NUMBER}"'"' -i mimoto/values.yaml

                        echo "After update:"
                        cat mimoto/values.yaml

                        git add mimoto/values.yaml
                        git commit -m "Auto-update image to ${DOCKER_IMAGE}" || echo "No changes to commit"
                        git push origin ${MANIFEST_BRANCH}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Updated image: ${env.DOCKER_IMAGE} in manifest repo."
        }
        failure {
            echo "Failed! Last attempted Docker Image Tag: ${env.DOCKER_IMAGE}"
        }
    }
}

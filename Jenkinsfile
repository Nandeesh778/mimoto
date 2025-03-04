pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/Nandeesh778/mimoto.git'
        GIT_BRANCH = 'test' 
        DOCKER_IMAGE_BASE = 'raparna154/inji-mimoto-service'
        MANIFEST_REPO = 'https://github.com/Aparnadeloitte/Inji-infra-azure.git'
        MANIFEST_BRANCH = 'main'
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:$PATH"
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
                        withEnv(["JAVA_HOME=${env.JAVA_HOME}", "PATH=${env.JAVA_HOME}/bin:${env.PATH}"]) {
                            sh """mvn clean package -DskipTests"""
                            sh """rm -f target/mimoto-*-javadoc.jar target/mimoto-*-sources.jar"""
                            sh """docker build -t ${env.DOCKER_IMAGE} ."""
                        }
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
                    withCredentials([usernamePassword(credentialsId: 'githubpat2', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                        rm -rf Inji-infra-azure
                        # Clone manifest repo
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Aparnadeloitte/Inji-infra-azure.git
                        cd Inji-infra-azure
                        git checkout ${MANIFEST_BRANCH}

                        echo "Before update:"
                        cat mimoto/values.yaml || true

                        # Force update the image tag
                        yq eval '.image.repository = "raparna154/inji-mimoto-service" |
                       .image.tag = "'"${env.COMMIT_HASH}-${env.BUILD_NUMBER}"'"
                        ' -i mimoto/values.yaml

                        # Debugging: Show after update
                        echo "After update:"
                        cat mimoto/values.yaml

                        # Commit & Push changes if there are any
                        git add mimoto/values.yaml
                        git commit -m "Auto-update image repository to ${DOCKER_IMAGE_BASE} and tag to ${env.IMAGE_TAG}" || echo "No changes to commit"
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

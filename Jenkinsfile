pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/Nandeesh778/mimoto.git'
        GIT_BRANCH = 'test' 
        DOCKER_IMAGE_BASE = 'raparna154/inji-mimoto-service'
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

        stage('Update Helm Values.yaml') {
            steps {
                script {
                    dir('mimoto') {  // Ensure we're in the correct repo directory
                        sh """
                        echo "Before update:"
                        cat values.yaml || true

                        # Update values.yaml using yq
                        yq eval '
                        .image.repository = "${DOCKER_IMAGE_BASE}" |
                        .image.tag = "${env.COMMIT_HASH}-${env.BUILD_NUMBER}"
                        ' -i values.yaml

                        # Debugging: Show after update
                        echo "After update:"
                        cat values.yaml

                        # Commit & Push changes if there are any
                        git add values.yaml
                        git commit -m "Auto-update image tag to ${env.COMMIT_HASH}-${env.BUILD_NUMBER}" || echo "No changes to commit"
                        git push origin ${GIT_BRANCH}
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

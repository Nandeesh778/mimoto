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
            

            // stage('Get Commit Hash') {
            //     steps {
            //         script {
            //             env.COMMIT_HASH = sh(
            //                 script: "cd mimoto && git rev-parse --short HEAD",
            //                 returnStdout: true
            //             ).trim()
            //             env.DOCKER_IMAGE = "${DOCKER_IMAGE_BASE}:${env.COMMIT_HASH}-${env.BUILD_NUMBER}"
            //             echo "Docker Image Tag: ${env.DOCKER_IMAGE}"
            //         }
            //     }
            // }

            // stage('Build Docker Image') {
            //     steps {
            //         script {
            //             dir('mimoto') {
            //                 withEnv(["JAVA_HOME=${env.JAVA_HOME}", "PATH=${env.JAVA_HOME}/bin:${env.PATH}"]) {
            //                     sh """mvn clean package -DskipTests"""
            //                     sh """rm -f target/mimoto-*-javadoc.jar target/mimoto-*-sources.jar"""
            //                     sh """docker build -t ${env.DOCKER_IMAGE} ."""
            //                 }
            //             }
            //         }
            //     }
            // }

            // stage('Push Docker Image') {
            //     steps {
            //         script {
            //             withCredentials([usernamePassword(credentialsId: 'dockerhubpat', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            //                 sh """
            //                 echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
            //                 docker push ${env.DOCKER_IMAGE}
            //                 docker logout
            //                 """
            //             }
            //         }
            //     }
            // }

        stage('Update Helm Values.yaml') {
            steps {
                script {
                    dir('mimoto/mimoto') {  // Ensure we're in the correct repo directory
                        sh """
                        echo "Before update:"
                        cat values.yaml || true

                        # Update values.yaml using yq
                        yq eval '
                        .image.repository = "${DOCKER_IMAGE_BASE}" |
                        .image.tag = "99441a3-6"
                        ' -i values.yaml

                        echo "After update:"
                        cat values.yaml
                        """
                    }
                }
            }
        }

        stage('Enter GitHub Token Manually') {
            steps {
                script {
                    env.GITHUB_TOKEN = input(
                        message: 'Enter your GitHub Personal Access Token:',
                        parameters: [password(defaultValue: '', description: 'GitHub Token', name: 'TOKEN')]
                    )
                }
            }
        }

        stage('Push Updated values.yaml') {
            steps {
                script {
                    dir('mimoto/mimoto') {
                        sh """
                        git config --global user.email "nandeeshv001@gmail.com"
                        git config --global user.name "Nandeesh778"
                        
                        git add values.yaml
                        git commit -m "Auto-update image tag to manifest repo" || echo "No changes to commit"

                        # Use the manually entered token
                        git push https://${env.GITHUB_TOKEN}@github.com/Nandeesh778/mimoto.git ${GIT_BRANCH}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Updated image: ${DOCKER_IMAGE_BASE}:99441a3-6"
        }
        failure {
            echo "Failed! Check error logs."
        }
    }
}

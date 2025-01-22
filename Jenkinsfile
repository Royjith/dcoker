pipeline {
    agent { label 'node-1' }  // Set the agent to use node-1

    environment {
        DOCKER_IMAGE = 'my-app'               // Docker image name
        DOCKER_TAG = 'jith'                   // Docker tag
        DOCKER_HUB_REPO = 'royjith/pikube'    // Docker Hub repository
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub'  // Docker Hub credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                git branch: 'main', credentialsId: 'dockerhub', url: 'https://github.com/Royjith/docker.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    def buildResult = sh(script: 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .', returnStatus: true)
                    if (buildResult != 0) {
                        error 'Docker build failed!'  // Explicitly fail if Docker build fails
                    }
                }
            }
        }

        // Uncomment if you want to run Trivy Scan:
        // stage('Trivy Scan') {
        //     steps {
        //         script {
        //             echo 'Clearing Trivy vulnerability database cache...'
        //             // Remove the Trivy DB cache directory if it exists
        //             sh 'rm -rf ~/.cache/trivy/db'
        //
        //             echo 'Running Trivy vulnerability scan on the Docker image...'
        //             // Run the scan without any skipped updates (no --skip-update or --skip-db-update)
        //             def scanResult = sh(script: 'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest', returnStatus: true)
        //             if (scanResult != 0) {
        //                 error 'Trivy scan failed!'  // Explicitly fail if Trivy scan fails
        //             }
        //         }
        //     }
        // }

        stage('Push Image to DockerHub') {
            steps {
                input message: 'Approve Deployment?', ok: 'Deploy'  // Manual approval for deployment
                script {
                    echo 'Pushing Docker image to DockerHub...'

                    try {
                        // Manually login to Docker Hub using the credentials
                        withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                        }

                        // Push the Docker image to Docker Hub
                        sh "docker push ${DOCKER_HUB_REPO}:${DOCKER_TAG}"

                    } catch (Exception e) {
                        error "Docker push failed: ${e.message}"  // Explicitly fail if push fails
                    }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after pipeline execution
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

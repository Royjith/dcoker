pipeline {
    agent { label 'node-1' }  // Set the agent to use node-1

    environment {
        DOCKER_IMAGE = 'my-app'              // Docker image name
        DOCKER_TAG = 'jith'                // Docker tag
        DOCKER_HUB_REPO = 'royjith/pikube'   // Docker Hub repository
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

        stage('Trivy Scan') {
            steps {
                script {
                    echo 'Running Trivy vulnerability scan on the Docker image...'

                    // Check if Trivy database exists (you may need to adjust the path based on your environment)
                    def trivyDbExists = fileExists('/root/.cache/trivy/db/trivy.db')  // Adjust the path if needed

                    // Run Trivy scan with or without --skip-db-update depending on the DB state
                    def trivyCommand = trivyDbExists ?
                        'trivy --severity HIGH,CRITICAL  --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_IMAGE}:${DOCKER_TAG}' :
                        'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_IMAGE}:${DOCKER_TAG}'

                    // Execute the Trivy command
                    def scanResult = sh(script: trivyCommand, returnStatus: true)

                    // Handle error if Trivy scan fails
                    if (scanResult != 0) {
                        error 'Trivy scan failed!'  // Explicitly fail if Trivy scan fails
                    }
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                input message: 'Approve Deployment?', ok: 'Deploy'  // Manual approval for deployment
                script {
                    echo 'Pushing Docker image to DockerHub...'
                    try {
                        docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}") {
                            def dockerImage = docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}")
                            dockerImage.push()  // Push the image with the tag defined in DOCKER_TAG variable
                        }
                    } catch (Exception e) {
                        error "Docker push failed: ${e.message}"  // Explicitly fail if push fails
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

pipeline {
    agent any

    environment {
        // Build & Docker Variables
        DOCKER_HUB_REPO = 'obedineflore536/petclinic'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        BUILD_TAG = "${BUILD_NUMBER}"
        
        // Git Variables
        GIT_CREDENTIALS_ID = 'github-token'
        GIT_REPO = 'https://github.com/Obedine-Flore/spring-petclinic-cicd.git'
        
        // Kubernetes Variables
        KUBECTL_CMD = '/usr/local/bin/kubectl'
        K8S_DEPLOYMENT_FILE = 'k8s-deployment.yaml'
        K8S_DEPLOYMENT_NAME = 'petclinic'
        K8S_NAMESPACE = 'default'
        
        // Docker Host is typically not needed for standard DIND/socket setups, removing for clarity.
        // If your agent environment requires it, uncomment: DOCKER_HOST = 'tcp://localhost:2375'
    }

    stages {
        stage('Checkout Source Code') {
            agent any
            steps {
                echo "=== Stage 1: Checking out code from GitHub ==="
                // Clean checkout ensures no lingering files from previous builds
                cleanWs()
                git branch: 'main', url: "${GIT_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }
        
        stage('Build Artifact (Maven)') {
            agent { label 'build-tools' }
            steps {
                echo "=== Stage 2: Compiling and packaging the Spring Boot application ==="
                // *** FIX: The project code is in the 'spring-petclinic' subdirectory. Navigate there. ***
                dir('spring-petclinic') {
                    // Use the 'container' step to execute Maven commands inside the 'maven' container
                    container('maven') { 
                        sh 'mvn clean package -DskipTests'
                    }
                }
                // Archive the artifact from the correct subdirectory
                archiveArtifacts artifacts: 'spring-petclinic/target/*.jar', fingerprint: true
            }
        }
        
        stage('Test') {
            agent { label 'build-tools' }
            steps {
                echo "=== Stage 3: Running Unit and Integration Tests ==="
                // *** FIX: Navigate to the project directory for tests. ***
                dir('spring-petclinic') {
                    // Use the 'container' step to execute Maven commands inside the 'maven' container
                    container('maven') { 
                        sh 'mvn test'
                        // Path is relative to the 'spring-petclinic' directory
                        junit 'target/surefire-reports/**/*.xml' 
                    }
                }
                echo "✅ All tests passed and results archived!"
            }
        }

        stage('Build Docker Image') {
            agent { label 'build-tools' } 
            steps {
                echo "=== Stage 4: Building Docker image using the packaged JAR ==="
                script {
                    container('maven') { 
                        // Check if the jar file is present in the subdirectory before attempting to build
                        sh "test -f spring-petclinic/target/*.jar || { echo 'ERROR: JAR artifact not found! Ensure Build stage ran successfully.'; exit 1; }"
                        
                        // Assumes the Dockerfile is in the repository root and uses the path 
                        // `spring-petclinic/target/*.jar` in its COPY command.
                        sh """
                            docker build -t ${DOCKER_HUB_REPO}:${BUILD_TAG} .
                            docker tag ${DOCKER_HUB_REPO}:${BUILD_TAG} ${DOCKER_HUB_REPO}:latest
                            docker images | grep petclinic
                        """
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            agent { label 'build-tools' }
            steps {
                echo "=== Stage 5: Pushing image to Docker Hub securely ==="
                script {
                    container('maven') {
                        // Use the declarative withRegistry wrapper for secure login/logout
                        docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                            sh "docker push ${DOCKER_HUB_REPO}:${BUILD_TAG}"
                            sh "docker push ${DOCKER_HUB_REPO}:latest"
                            echo "✅ Images pushed successfully!"
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            agent any
            steps {
                echo "=== Stage 6: Deploying to Kubernetes cluster ==="
                script {
                    // 1. Temporarily replace the image tag in the deployment file
                    sh "sed -i 's|${DOCKER_HUB_REPO}:.*|${DOCKER_HUB_REPO}:${BUILD_TAG}|g' ${K8S_DEPLOYMENT_FILE}"
                    sh "echo 'Updated deployment file with image tag: ${DOCKER_HUB_REPO}:${BUILD_TAG}'"
                    
                    // 2. Apply the deployment
                    sh "${KUBECTL_CMD} apply -f ${K8S_DEPLOYMENT_FILE}"
                    
                    // 3. Wait for rollout
                    echo "Waiting for deployment ${K8S_DEPLOYMENT_NAME} to complete in namespace ${K8S_NAMESPACE}..."
                    sh "${KUBECTL_CMD} rollout status deployment/${K8S_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=5m"
                    
                    echo "✅ Deployment successful! Final status:"
                    sh "${KUBECTL_CMD} get pods -n ${K8S_NAMESPACE} -l app=${K8S_DEPLOYMENT_NAME}"
                    sh "${KUBECTL_CMD} get svc ${K8S_DEPLOYMENT_NAME}-service -n ${K8S_NAMESPACE}"
                }
            }
        }
    }
    
    post {
        success {
            echo '========================================='
            echo '✅ Pipeline completed successfully!'
            echo '========================================='
            echo 'Application is accessible at:'
            echo 'http://<NODE_IP>:30080'
            echo '========================================='
        }
        failure {
            echo '========================================='
            echo '❌ Pipeline failed!'
            echo 'Check the console output for errors.'
            echo '========================================='
        }
        // 'always' block is no longer strictly needed for logout due to 'withRegistry' wrapper, 
        // but can be used for general cleanup.
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
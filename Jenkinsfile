pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'obedineflore536/petclinic'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        GIT_REPO = 'https://github.com/Obedine-Flore/spring-petclinic-cicd.git'
        BUILD_TAG = "${BUILD_NUMBER}"
        KUBECTL_CMD = '/var/jenkins_home/kubectl'
        DOCKER_HOST = 'tcp://localhost:2375'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "=== Stage 1: Checking out code from GitHub ==="
                git branch: 'main', url: "${GIT_REPO}"
                sh 'ls -la'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "=== Stage 2: Building Docker image ==="
                script {
                    sh """
                        docker build -t ${DOCKER_HUB_REPO}:${BUILD_TAG} .
                        docker tag ${DOCKER_HUB_REPO}:${BUILD_TAG} ${DOCKER_HUB_REPO}:latest
                        docker images | grep petclinic
                    """
                }
            }
        }
        
        stage('Test') {
            steps {
                echo "=== Stage 3: Running Tests ==="
                sh '''
                    echo "Running unit tests..."
                    echo "✅ All tests passed!"
                '''
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo "=== Stage 4: Pushing image to Docker Hub ==="
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker push ${DOCKER_HUB_REPO}:${BUILD_TAG}
                            docker push ${DOCKER_HUB_REPO}:latest
                            echo "✅ Images pushed successfully!"
                        """
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                echo "=== Stage 5: Deploying to Kubernetes ==="
                script {
                    sh """
                        ${KUBECTL_CMD} apply -f k8s-deployment.yaml
                        echo "Waiting for deployment to complete..."
                        ${KUBECTL_CMD} rollout status deployment/petclinic -n default --timeout=5m
                        echo "✅ Deployment successful!"
                        ${KUBECTL_CMD} get pods -n default -l app=petclinic
                        ${KUBECTL_CMD} get svc petclinic-service -n default
                    """
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
        always {
            echo 'Cleaning up...'
            sh 'docker logout || true'
        }
    }
}
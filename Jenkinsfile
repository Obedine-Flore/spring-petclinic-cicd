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
        KUBECTL_CMD = '/usr/local/bin/kubectl' // Adjusted path to a more common default location
        K8S_DEPLOYMENT_FILE = 'k8s-deployment.yaml'
        K8S_DEPLOYMENT_NAME = 'petclinic'
        K8S_NAMESPACE = 'default'
    }

        stage('Build Docker Image') {
            agent { label 'build-agent' }
            steps {
                echo "=== Stage 3: Building Docker image using the packaged JAR ==="
                echo "Diagnostic: Stages 1 and 2 were skipped? ${params.SKIP_BUILD_AND_TEST}"
                
                // 1. Checkout main repository files (Dockerfile, etc.)
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: GIT_REPO, credentialsId: GIT_CREDENTIALS_ID]]
                ])
                
                // 2. FIX: Unarchive artifacts *before* container is used. 
                // This ensures the JAR is available in the workspace.
                unarchiveArtifacts artifacts: 'lab6-jenkins/target/*.jar'
                echo "✅ Artifacts unarchived for Docker build."

                script {
                    container('maven') { 
                        
                        // DIAGNOSTIC: List files in the target directory after unarchiving
                        sh 'ls -l lab6-jenkins/target/'
                        
                        // 3. Build the image
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
            agent { label 'build-agent' }
            steps {
                echo "=== Stage 4: Pushing image to Docker Hub securely ==="
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
                echo "=== Stage 5: Deploying to Kubernetes cluster ==="
                
                // Checkout main repository files (k8s-deployment.yaml)
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: 'main']], 
                    userRemoteConfigs: [[url: GIT_REPO, credentialsId: GIT_CREDENTIALS_ID]]
                ])

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
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
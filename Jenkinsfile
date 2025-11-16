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
                echo "=== Stage 2: Compiling and packaging the Spring Boot application (Diagnostic Run) ==="

                // --- DIAGNOSTIC STEP: Check workspace contents to find the POM ---
                // *** IMPORTANT: Share the output of the 'ls -F' command below ***
                container('maven') {
                    sh 'echo "--- Workspace Root Contents (Looking for the app folder) ---"; ls -F'
                    sh 'echo "------------------------------------------------------------"'
                }
                
                // *** COMMENTING OUT FAILING STEPS - WE NEED THE DIRECTORY NAME FIRST ***
                /* dir('spring-petclinic') {
                    container('maven') { 
                        sh 'mvn clean package -DskipTests'
                    }
                }
                archiveArtifacts artifacts: 'spring-petclinic/target/*.jar', fingerprint: true
                */
            }
        }
        
        stage('Test') {
            // Stage skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 3: Running Unit and Integration Tests - SKIPPED"
            }
        }

        stage('Build Docker Image') {
            // Stage skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 4: Building Docker image - SKIPPED"
            }
        }
        
        stage('Push to Docker Hub') {
            // Stage skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 5: Pushing image to Docker Hub - SKIPPED"
            }
        }
        
        stage('Deploy to Kubernetes') {
            // Stage skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 6: Deploying to Kubernetes cluster - SKIPPED"
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
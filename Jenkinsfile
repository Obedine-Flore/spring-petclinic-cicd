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
                echo "=== Stage 2: DIAGNOSTIC: Locating the pom.xml inside 'lab6-jenkins' ==="

                // --- NEW DIAGNOSTIC STEP: Recursively check contents of lab6-jenkins ---
                // *** IMPORTANT: Share the complete output of the 'ls -RF' command below ***
                dir('lab6-jenkins') {
                    container('maven') {
                        sh 'echo "--- Contents of lab6-jenkins/ (Recursive List) ---"; ls -RF'
                        sh 'echo "------------------------------------------------------------"'
                    }
                }
                
                // *** FAILING/SUBSEQUENT STEPS ARE COMMENTED OUT AGAIN TO GET DIAGNOSTIC OUTPUT ***
                /*
                // Use the 'container' step to execute Maven commands inside the 'maven' container
                // sh 'mvn clean package -DskipTests'
                // Archive the artifact from the correct subdirectory (path relative to workspace root)
                // archiveArtifacts artifacts: 'lab6-jenkins/target/*.jar', fingerprint: true
                */
            }
        }
        
        stage('Test') {
            // Stages skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 3: Running Unit and Integration Tests - SKIPPED FOR DIAGNOSTIC"
            }
        }

        stage('Build Docker Image') {
            // Stages skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 4: Building Docker image - SKIPPED FOR DIAGNOSTIC"
            }
        }
        
        stage('Push to Docker Hub') {
            // Stages skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 5: Pushing image to Docker Hub - SKIPPED FOR DIAGNOSTIC"
            }
        }
        
        stage('Deploy to Kubernetes') {
            // Stages skipped temporarily until build succeeds
            agent any
            steps {
                echo "Stage 6: Deploying to Kubernetes cluster - SKIPPED FOR DIAGNOSTIC"
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
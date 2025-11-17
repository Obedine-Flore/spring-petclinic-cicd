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
                echo "=== Stage 2: DIAGNOSTIC: FINDING THE EXACT LOCATION OF pom.xml ==="

                // --- NEW DIAGNOSTIC STEP: Recursively search for pom.xml from workspace root ---
                // *** IMPORTANT: Share the complete output of the 'find' command below ***
                container('maven') { 
                    sh 'echo "--- Searching for pom.xml from Workspace Root ---"; find . -name "pom.xml"'
                    sh 'echo "------------------------------------------------------------"'
                }
                
                // *** FAILING/SUBSEQUENT STEPS ARE COMMENTED OUT AGAIN TO GET DIAGNOSTIC OUTPUT ***
                /*
                // dir('lab6-jenkins/lab6-jenkins-2') {
                //     container('maven') { 
                //         sh 'mvn clean package -DskipTests'
                //     }
                // }
                // archiveArtifacts artifacts: 'lab6-jenkins/lab6-jenkins-2/target/*.jar', fingerprint: true
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
            echo '✅ Pipeline completed successfully (FOURTH DIAGNOSTIC RUN)!'
            echo '========================================='
        }
        failure {
            echo '========================================='
            echo '❌ Pipeline failed! (FOURTH DIAGNOSTIC RUN)'
            echo 'Check the console output for errors.'
            echo '========================================='
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
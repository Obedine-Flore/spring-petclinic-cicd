pipeline {
    agent {
        kubernetes {
            // This YAML defines the "Build Pod" that will live only for the duration of the build
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.8.5-openjdk-17
                command: ['cat']
                tty: true
              - name: docker
                image: docker:latest
                command: ['cat']
                tty: true
                volumeMounts:
                - name: dockersock
                  mountPath: /var/run/docker.sock
              - name: kubectl
                image: bitnami/kubectl:latest
                command: ['cat']
                tty: true
              volumes:
              - name: dockersock
                hostPath:
                  path: /var/run/docker.sock
            '''
        }
    }

    environment {
        // Build & Docker Variables
        DOCKER_HUB_REPO = 'obedineflore536/petclinic'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        BUILD_TAG = "${BUILD_NUMBER}"
        
        // Git Variables
        GIT_CREDENTIALS_ID = 'github-token'
        GIT_REPO = 'https://github.com/Obedine-Flore/spring-petclinic-cicd.git'
        
        // Kubernetes Variables
        KUBECTL_CMD = 'kubectl' 
        K8S_DEPLOYMENT_FILE = 'k8s-deployment.yaml'
        K8S_DEPLOYMENT_NAME = 'petclinic'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Build Artifact (Maven)') {
            steps {
                container('maven') {
                    echo "=== Stage 1: Compiling and packaging (Using Maven Container) ==="
                    
                    // 1. Explicit Checkout
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[url: GIT_REPO, credentialsId: GIT_CREDENTIALS_ID]],
                        extensions: [[$class: 'SubmoduleOption', disableSubmodules: false, recursiveSubmodules: true, parentCredentials: true]]
                    ])
                    echo "✅ Repository cloned."

                    // 2. Initialize Submodules
                    sh 'git submodule update --init --recursive'
                    
                    // 3. Run Maven build
                    dir('lab6-jenkins') {
                        sh 'mvn clean package -DskipTests'
                    }
                    
                    // Archive the artifact
                    archiveArtifacts artifacts: 'lab6-jenkins/target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Test') {
            steps {
                container('maven') {
                    echo "=== Stage 2: Running Tests ==="
                    dir('lab6-jenkins') {
                        sh 'mvn test'
                        junit 'target/surefire-reports/**/*.xml'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // SWITCHING TO DOCKER CONTAINER
                container('docker') {
                    echo "=== Stage 3: Building Docker image ==="
                    script {
                        dir('lab6-jenkins') {
                            sh "ls -l target/*.jar"
                            sh "docker build -t ${DOCKER_HUB_REPO}:${BUILD_TAG} ."
                            sh "docker tag ${DOCKER_HUB_REPO}:${BUILD_TAG} ${DOCKER_HUB_REPO}:latest"
                        }
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    echo "=== Stage 4: Pushing to Docker Hub ==="
                    script {
                      // FIX: Use the 'credentialsId:' keyword
                        docker.withRegistry('https://registry.hub.docker.com', credentialsId: 'dockerhub-credentials') {
                            sh "docker push ${DOCKER_HUB_REPO}:${BUILD_TAG}"
                            sh "docker push ${DOCKER_HUB_REPO}:latest"
                        }
                    }
                }   
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                // SWITCHING TO KUBECTL CONTAINER
                container('kubectl') {
                    echo "=== Stage 5: Deploying to Kubernetes ==="
                    script {
                        // Update manifest with new tag
                        sh "sed -i 's|${DOCKER_HUB_REPO}:.*|${DOCKER_HUB_REPO}:${BUILD_TAG}|g' ${K8S_DEPLOYMENT_FILE}"
                        
                        // Apply deployment
                        sh "${KUBECTL_CMD} apply -f ${K8S_DEPLOYMENT_FILE}"
                        
                        // Wait for rollout
                        sh "${KUBECTL_CMD} rollout status deployment/${K8S_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=5m"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        always {
            sh "chmod -R a+rwx ${WORKSPACE}"
            deleteDir()
        }
    }
}

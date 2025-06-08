pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username/organization
        DOCKER_REGISTRY = 'https://docker.io'
        DOCKER_IMAGE_NAME = 'delaroth/my-kubernetes-app' // e.g., mydockerusername/my-node-app
        HELM_CHART_PATH = 'levis-nginx' // Path to your Helm chart relative to repo root
        KUBERNETES_NAMESPACE = 'levi' // Or your specific Kubernetes namespace
        RELEASE_NAME = 'levi-website' // Helm release name
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def gitCommit = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    // Use a timestamp or Git commit hash as the tag
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${gitCommit}"
                    echo "Building Docker image: ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
                    docker.build("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}", "-f Dockerfile .")
                }
            }
        }

       stage('Push Docker Image') {
                steps {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            echo "Attempting Docker login within pipeline..."
                            // Explicit Docker login using the credentials provided by Jenkins
                            // Use --password-stdin to avoid exposing the password in logs
                            sh """
                                echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin ${DOCKER_REGISTRY}
                            """
                            echo "Docker login attempt finished. Now pushing image..."
                            
                            // Original push command, now that login is explicit
                            docker.image("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}").push()
                        }
                    }
                }
            }

stage('Deploy to Kubernetes with Helm') {
    steps {
        script {
            withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG_FILE_PATH')]) {
                // Now, use withEnv to expose KUBECONFIG_FILE_PATH as KUBECONFIG
                withEnv(["KUBECONFIG=${KUBECONFIG_FILE_PATH}"]) { 
                    echo "Deploying with Helm to Kubernetes namespace: ${KUBERNETES_NAMESPACE}"

                    // No need for 'export KUBECONFIG' inside sh anymore
                    sh """
                        helm upgrade --install ${RELEASE_NAME} ${HELM_CHART_PATH} \\
                            --namespace ${KUBERNETES_NAMESPACE} \\
                            --set image.tag=${IMAGE_TAG}
                    """
                    echo "Helm deployment initiated. Check Kubernetes for rollout status."
                } 
            }
        }
    }
}

// ... (rest of your Jenkinsfile)

    }

    post {
        always {
            cleanWs() // Clean up workspace after build
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed. Check logs for details.'
        }
    }
}

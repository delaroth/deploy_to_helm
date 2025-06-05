pipeline {
    agent {
        label 'helm deployer' 
    }

    environment {
        // Replace with your Docker Hub username/organization
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE_NAME = 'delaroth/my-kubernetes-app' // e.g., mydockerusername/my-node-app
        HELM_CHART_PATH = 'levis_nginx' // Path to your Helm chart relative to repo root
        KUBERNETES_NAMESPACE = 'default' // Or your specific Kubernetes namespace
        RELEASE_NAME = 'levi_website' // Helm release name
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
                    // Use the Jenkins credential ID for your Docker Hub login
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        docker.withRegistry("${DOCKER_REGISTRY}", 'DockerHub') {
                            echo "Pushing Docker image: ${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
                            docker.image("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}").push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                script {
                    echo "Deploying with Helm to Kubernetes namespace: ${KUBERNETES_NAMESPACE}"

                    // Ensure Helm is installed and configured on the agent
                    // Update the Helm chart's values.yaml with the new image tag
                    // This is one way; you can also pass --set arguments to helm upgrade
                    sh "helm upgrade --install ${RELEASE_NAME} ${HELM_CHART_PATH} " +
                       "--namespace ${KUBERNETES_NAMESPACE} " +
                       "--set image.tag=${IMAGE_TAG}"

                    echo "Helm deployment initiated. Check Kubernetes for rollout status."
                }
            }
        }
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

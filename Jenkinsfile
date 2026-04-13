pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'virendranawkar'
        IMAGE_NAME     = 'todo-app'
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test') {
            steps {
		sh 'npm install'
        	sh 'npm test -- --watchAll=false --passWithNoTests'
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh "echo $DH_PASS | docker login -u $DH_USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                withCredentials([string(credentialsId: 'aks-kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
                    sh """
                        echo "$KUBECONFIG_CONTENT" > /tmp/aks-kubeconfig
                        sed -i 's|IMAGE_PLACEHOLDER|${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yaml
                        kubectl --kubeconfig=/tmp/aks-kubeconfig apply -f k8s/
                        sed -i 's|${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|IMAGE_PLACEHOLDER|g' k8s/deployment.yaml
                        rm -f /tmp/aks-kubeconfig
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'App deployed successfully to AKS!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}

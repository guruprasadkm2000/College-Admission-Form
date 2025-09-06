pipeline {
    agent any

    // Parameters to select branch, Docker tag, namespace
    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Git branch to build from')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        choice(name: 'K8S_NAMESPACE', choices: ['default', 'dev', 'staging', 'prod'], description: 'Kubernetes namespace for deployment')
        string(name: 'APP_PORT', defaultValue: '3028', description: 'Port your Node.js app listens on')
    }

    environment {
        DOCKERHUB_USER = 'guruprasadkm2000'
        DOCKERHUB_PASS = credentials('dockerhub-creds')
        AWS_REGION = 'us-east-1'
        CLUSTER_NAME = 'CAF-Cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}", url: 'https://github.com/guruprasadkm2000/College-Admission-Form.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKERHUB_USER/college-admission:${params.IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin"
                    sh "docker push $DOCKERHUB_USER/college-admission:${params.IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Update kubeconfig
                    sh "aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME"

                    // Replace Docker image and PORT in deployment.yaml
                    sh """
                        sed -i 's|image:.*|image: $DOCKERHUB_USER/college-admission:${params.IMAGE_TAG}|g' k8s/deployment.yaml
                        sed -i 's|containerPort:.*|containerPort: ${params.APP_PORT}|g' k8s/deployment.yaml
                        sed -i 's|targetPort:.*|targetPort: ${params.APP_PORT}|g' k8s/deployment.yaml
                    """

                    // Apply manifests in selected namespace
                    sh "kubectl apply -n ${params.K8S_NAMESPACE} -f k8s/mongodb-deployment.yaml"
                    sh "kubectl apply -n ${params.K8S_NAMESPACE} -f k8s/deployment.yaml"
                }
            }
        }
    }
}


pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Git branch to build from')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        choice(name: 'K8S_NAMESPACE', choices: ['default', 'dev', 'staging', 'prod'], description: 'Kubernetes namespace for deployment')
    }

    environment {
        DOCKERHUB_USER = 'guruprasadkm2000'
        DOCKERHUB_PASS = credentials('dockerhub-creds') // Jenkins credentials
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

        stage('Push to DockerHub') {
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
                    // Update kubeconfig to talk to EKS
                    sh "aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME"

                    // Replace image tag dynamically in deployment.yaml before applying
                    sh """
                      sed -i 's|<your-dockerhub-username>/college-admission:latest|$DOCKERHUB_USER/college-admission:${params.IMAGE_TAG}|g' k8s/deployment.yaml
                    """

                    // Apply manifests to chosen namespace
                    sh "kubectl apply -n ${params.K8S_NAMESPACE} -f k8s/mongodb-deployment.yaml"
                    sh "kubectl apply -n ${params.K8S_NAMESPACE} -f k8s/deployment.yaml"
                }
            }
        }
    }
}

pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
    }

    tools {
        maven 'MAVEN'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/prashanth-singaravel/devops-automation.git']]
                ])
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t prashanthsingaravel/devops-integration .'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'prashanthsingaravel-dockerhub', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u prashanthsingaravel -p ${dockerhubpwd}'
                }
                sh 'docker push prashanthsingaravel/devops-integration'
            }
        }

        stage('Check kubectl and workspace') {
            steps {
                // Verify kubectl is available
                sh 'echo "PATH is: $PATH"'
                sh 'which kubectl'
                sh 'kubectl version --client'

                // List files in workspace to confirm deployment.yaml exists
                sh 'ls -l deploymentservice.yaml'
            }
        }

        stage('Deploy to k8s') {
            steps {
                script {
                    // Wrap deployment in try-catch to print errors clearly
                    try {
                        sh 'kubectl apply -f deploymentservice.yaml'
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.getMessage()}"
                        error("kubectl apply command failed, aborting pipeline")
                    }
                }
            }
        }
    }
}

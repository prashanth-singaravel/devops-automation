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
                    userRemoteConfigs: [[url: 'https://github.com/Raj9556/devops-automation.git']]
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
                sh 'docker build -t srk96/devops-integration .'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'srk96', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u srk96 -p ${dockerhubpwd}'
                }
                sh 'docker push srk96/devops-integration'
            }
        }

        stage('Check kubectl and workspace') {
            steps {
                // Verify kubectl is available
                sh 'echo "PATH is: $PATH"'
                sh 'which kubectl'
                sh 'kubectl version --client'

                // List files in workspace to confirm deployment.yaml exists
                sh 'ls -l deployment.yaml'
            }
        }

        stage('Deploy to k8s') {
            steps {
                script {
                    // Wrap deployment in try-catch to print errors clearly
                    try {
                        sh 'kubectl apply -f deployment.yaml'
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.getMessage()}"
                        error("kubectl apply command failed, aborting pipeline")
                    }
                }
            }
        }
    }
}

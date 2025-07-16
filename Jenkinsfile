pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
    }

    tools {
        maven 'MAVEN'
    }

    stages {
        stage('Build Maven') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Raj9556/devops-automation.git']]])
                sh 'mvn clean install'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t srk96/devops-integration .'
                }
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

        stage('Deploy to k8s') {
            steps {
                script {
                    kubernetesDeploy(
                        configs: 'deploymentservice.yaml',
                        kubeconfigId: 'k8sconfigpwd'
                    )
                }
            }
        }
    }
}

pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the code from the specified SCM
                git url: 'https://github.com/Renukadema/Calculator.git', branch: 'master'
            }
        }

        stage('Build with Maven') {
            steps {
                // Execute Maven build
                sh 'mvn clean install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube', credentialsId: 'sonar-token') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=Calculator -Dsonar.projectName='Calculator'"
                }    
            } 
        }
        stage('Build docker') {
            steps {
                //sh "sudo docker build -t calculator . --file dockerfile"
                sh '''#!/bin/bash
                docker build -t calculator . --file Dockerfile
                '''
            }
        }
        stage('helm Package') {
            steps {
                //sh "sudo docker build -t calculator . --file dockerfile"
                sh '''#!/bin/bash
                helm create nginx
                helm lint nginx
                helm package nginx
                '''
            }
        }
        stage('Namespace Creation') {
            steps {
                script {
                    // Load AWS credentials from Jenkins
                    withAWS(credentials: 'aws_config') {
                        // Update kubeconfig to connect to the EKS cluster
                        sh "aws eks update-kubeconfig --name demo-eks --region us-east-1"
                        sleep 10
                        sh "kubectl create ns gmail"
                        sleep 5
                        sh "kubectl get ns"
                    }
                }
            }
        }
        stage('Connect to AWS and EKS') {
            steps {
                script {
                    // Load AWS credentials from Jenkins
                    withAWS(credentials: 'aws_config') {
                        // Update kubeconfig to connect to the EKS cluster
                        sh "aws eks update-kubeconfig --name demo-eks --region us-east-1"
                        sh "kubectl apply -f deployment.yaml -n gmail"
                        sh "kubectl apply -f nginx-loadbalancer.yaml -n gmail"
                    }
                }
            }
        }
        
        stage('Verify Application Deploy') {
            steps {
                script {
                    // Load AWS credentials from Jenkins
                    withAWS(credentials: 'aws_config') {
                        // Update kubeconfig to connect to the EKS cluster
                        sh "aws eks update-kubeconfig --name demo-eks --region us-east-1"
                        sh "kubectl get nodes"
                        sh "kubectl get all -n gmail"
                        sh "kubectl delete ns test"
                        sh "kubectl delete ns facebook"
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}

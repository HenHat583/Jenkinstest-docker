#!/usr/bin/env groovy

pipeline {
    agent any

    stages {
        stage('Cleanup') {
            steps {
                sh 'echo "Performing cleanup..."'
                sh 'rm -rf flask flask.tar.gz'
            }
        }

        stage('Clone') {
            steps {
                sh 'echo "Building..."'
                sh 'git clone https://github.com/HenHat583/flask.git'
                sh 'ls flask'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'echo "Building Docker image..."'
                sh 'sudo docker build -t henhat583/flask-app:latest ./flask'
            }
        }
        
        stage('Push To Docker Hub') {
            steps {
                sh 'echo "Pushing to Docker Hub..."'
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                sh 'sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                sh 'sudo docker push henhat583/flask-app:latest'
                }
            }
        }

        stage('Copy from S3 to EC2') {
            steps {
                sh 'echo "Copying S3 object to EC2..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'sudo scp -i /home/henhat583/.ssh/hen.pem -o StrictHostKeyChecking=no flask.tar.gz ec2-user@16.16.251.250:/home/ec2-user'
                }
            }
        }

        stage('Run Flask App on EC2') {
            steps {
                sh 'echo "Running Flask app on EC2..."'
                sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@16.16.251.250 "cd /home/ec2-user && tar xvf flask.tar.gz"'
                withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                    sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@16.16.251.250 "sudo bash -x flask/deploy.sh"'
                }
            }
        }
    }
}

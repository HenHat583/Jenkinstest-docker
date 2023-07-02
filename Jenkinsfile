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

        stage('Build & Zip') {
            steps {
                script {
                    sh 'tar czvf flask.tar.gz flask'
                }
            }
        }

        stage('Push To Cloud') {
            steps {
                sh 'echo "Pushing to S3..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'aws s3 cp flask.tar.gz s3://hensbucket/flask.tar.gz'
                }
            }
        }

        stage('Copy from S3 to EC2') {
            steps {
                sh 'echo "Copying S3 object to EC2..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'sudo scp -i /home/henhat583/.ssh/hen.pem -o StrictHostKeyChecking=no flask.tar.gz ec2-user@13.49.138.51:/home/ec2-user'
                }
            }
        }

        stage('Run Flask App on EC2') {
            steps {
                sh 'echo "Running Flask app on EC2..."'
                sh 'sudo ssh -i /home/henhat583/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.49.138.51 "cd /home/ec2-user && tar xvf flask.tar.gz"'
                sh 'sudo ssh -i /home/henhat583/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.49.138.51 "export FLASK_APP=/home/ec2-user/flask/app.py && flask run"'
            }
        }
    }
}

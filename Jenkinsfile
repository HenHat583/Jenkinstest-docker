pipeline {
    agent any
    
    stages {
        stage('Cleanup') {
            steps {
                sh 'echo Performing cleanup...'
                sh 'rm -rf flask flask.tar.gz'
            }
        }
        
        stage('Clone') {
            steps {
                sh 'echo Building...'
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
                sh 'echo Pushing to S3...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'aws s3 cp flask.tar.gz s3://hensbucket/flask.tar.gz'
                }
            }
        }
        
        stage('Copy for S3 to EC2') {
            steps {
                sh 'echo Copying S3 object to EC2...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'scp -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no flask.tar.gz ec2-user@13.51.44.20:/home/ec2-user'
                }
            }
        }
    }
}

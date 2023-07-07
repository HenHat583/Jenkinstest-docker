pipeline {
    agent any

    environment {
        testInstance = '16.16.193.2'
        prodInstance = '13.53.133.249'
        sshKeyPath = '/var/lib/jenkins/.ssh/hen.pem'
        dockerImageName = 'henhat583/flask-app:latest'
        flaskAppPath = '/flask'
    }

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
                sh 'sudo docker build -t $dockerImageName ./flask'
            }
        }

        stage('Push To Docker Hub') {
            steps {
                sh 'echo "Pushing to Docker Hub..."'
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh 'sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
                    sh 'sudo docker push $dockerImageName'
                }
            }
        }

        stage('Pull Docker Image on Test Server') {
            steps {
                sh 'echo "Pulling Docker image on test server..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$testInstance \"docker pull $dockerImageName\""
                }
            }
        }

        stage('Check Flask with cURL on test server') {
            steps {
                sh 'echo "Building and running Flask app on the test server..."'
                sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$testInstance \"sudo docker rm -f test""
                sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$testInstance \"sudo docker run -d -p 5000:5000 --name test $dockerImageName\""
                sh 'sleep 15' // Give some time for the app to start

                sh 'echo "Checking Flask app using cURL..."'
                sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$testInstance \"curl -s http://$testInstance:5000\""
            }
            post {
                success {
                    echo "Flask app is running successfully on the test server."
                }
                failure {
                    error "Flask app check on the test server failed. Exiting the pipeline."
                }
            }
        }

        stage('Pull Docker Image on EC2') {
            steps {
                sh 'echo "Pulling Docker image on EC2 prod server..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$prodInstance \"docker pull $dockerImageName\""
                }
            }
        }

        
        stage('Run Flask App on EC2 prod server') {
            steps {
                sh 'echo "Running Flask app on EC2 prod server..."'
                sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$prodInstance \"sudo docker rm -f prod""
                sh "sudo ssh -i $sshKeyPath -o StrictHostKeyChecking=no ec2-user@$prodInstance \"sudo docker run -d -p 5000:5000 --name prod $dockerImageName\""
            }
        }
    }
}

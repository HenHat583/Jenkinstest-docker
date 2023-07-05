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

        stage('Pull Docker Image on EC2') {
            steps {
                sh 'echo "Pulling Docker image on EC2..."'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'henhat583']]) {
                    sh 'sudo ssh -i /home/henhat583/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.50.231.2 "docker pull henhat583/flask-app:latest"'
                }
            }
        }

        stage('Check Flask with cURL on a test server') {
            steps {
                sh 'echo "Building and running Flask app on the test server..."'
                sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.53.48.43 "cd /flask && sudo docker build -t test/flask-app:latest . && sudo docker run -d -p 5000:5000 test/flask-app:latest"'
                sh 'sleep 10' // Give some time for the app to start

                sh 'echo "Checking Flask app using cURL..."'
                sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.53.48.43 "curl -s http://localhost:5000"'
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

        stage('Run Flask App on EC2 prod server') {
            steps {
                sh 'echo "Running Flask app on EC2 prod server..."'
                sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.50.231.2 "cd /home/ec2-user && tar xvf flask.tar.gz"'
                withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                    sh 'ssh -i /var/lib/jenkins/.ssh/hen.pem -o StrictHostKeyChecking=no ec2-user@13.50.231.2 "sudo bash -x flask/deploy.sh"'
                }
            }
        }
    }
}

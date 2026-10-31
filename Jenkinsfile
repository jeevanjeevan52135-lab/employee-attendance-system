pipeline {
    agent any

    environment {
        IMAGE = 'YOUR_DOCKERHUB_USERNAME/employee-app'
        SERVER = 'ubuntu@YOUR_EC2_PUBLIC_IP'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                url: 'YOUR_GITHUB_REPOSITORY_URL'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE:$BUILD_NUMBER .'
            }
        }

        stage('Test') {
            steps {
                sh 'python3 -m compileall -q .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" |
                        docker login -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE:$BUILD_NUMBER
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes $SERVER "
                        docker pull $IMAGE:$BUILD_NUMBER &&
                        docker stop employee-app || true
                        docker rm employee-app || true
                        docker run -d --name employee-app \
                        -p 80:5000 $IMAGE:$BUILD_NUMBER
                        "
                    '''
                }
            }
        }
    }
}
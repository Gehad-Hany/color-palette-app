pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'geha023'
        IMAGE_NAME = 'color-palette'
        CONTAINER_NAME = 'palette-app'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling code from GitHub...'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$BUILD_NUMBER .'
                sh 'docker tag $DOCKER_HUB_USER/$IMAGE_NAME:$BUILD_NUMBER $DOCKER_HUB_USER/$IMAGE_NAME:latest'
            }
        }

        stage('Test Application') {
            steps {
                sh 'docker rm -f test-palette-app || true'
                sh 'docker run -d --name test-palette-app -p 5050:5000 $DOCKER_HUB_USER/$IMAGE_NAME:$BUILD_NUMBER'
                sh 'sleep 10'
                sh 'docker exec test-palette-app python3 -c "import urllib.request; urllib.request.urlopen(\'http://localhost:5000/health\')"'
                sh 'docker rm -f test-palette-app'
                echo 'Health check passed!'
            }
        }

        stage('Login and Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:$BUILD_NUMBER'
                    sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:latest'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker rm -f $CONTAINER_NAME || true'
                sh 'docker run -d --name $CONTAINER_NAME -p 5000:5000 $DOCKER_HUB_USER/$IMAGE_NAME:latest'
                echo 'Deployment successful! Access at http://localhost:5000'
            }
        }
    }
}
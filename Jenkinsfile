pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'gabrielamand'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:GabrielAmand/jenkins-exam-devops.git'
            }
        }

        stage('Build movie-service image') {
            steps {
                sh 'docker build -t $DOCKERHUB_USERNAME/movie-service:latest -t $DOCKERHUB_USERNAME/movie-service:$IMAGE_TAG ./movie-service'
            }
        }

        stage('Build cast-service image') {
            steps {
                sh 'docker build -t $DOCKERHUB_USERNAME/cast-service:latest -t $DOCKERHUB_USERNAME/cast-service:$IMAGE_TAG ./cast-service'
            }
        }

        stage('Push images to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKERHUB_USERNAME/movie-service:latest
                        docker push $DOCKERHUB_USERNAME/movie-service:$IMAGE_TAG
                        docker push $DOCKERHUB_USERNAME/cast-service:latest
                        docker push $DOCKERHUB_USERNAME/cast-service:$IMAGE_TAG
                    '''
                }
            }
        }
    }
}
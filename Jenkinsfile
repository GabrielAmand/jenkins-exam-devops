pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'gabrielamand'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG = '/etc/rancher/k3s/k3s.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:GabrielAmand/jenkins-exam-devops.git'
            }
        }

        stage('Build images') {
            steps {
                sh '''
                    docker build -t $DOCKERHUB_USERNAME/movie-service:latest -t $DOCKERHUB_USERNAME/movie-service:$IMAGE_TAG ./movie-service
                    docker build -t $DOCKERHUB_USERNAME/cast-service:latest -t $DOCKERHUB_USERNAME/cast-service:$IMAGE_TAG ./cast-service
                '''
            }
        }

        stage('Push images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
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

        stage('Deploy to dev') {
            steps {
                sh 'helm upgrade --install movie-platform-dev ./charts -n dev -f ./charts/values-dev.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml'
            }
        }

        stage('Deploy to qa') {
            steps {
                sh 'helm upgrade --install movie-platform-qa ./charts -n qa -f ./charts/values-qa.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml'
            }
        }

        stage('Deploy to staging') {
            steps {
                sh 'helm upgrade --install movie-platform-staging ./charts -n staging -f ./charts/values-staging.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml'
            }
        }

        stage('Deploy to prod') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to PRODUCTION?', ok: 'Deploy'

                sh 'helm upgrade --install movie-platform-prod ./charts -n prod -f ./charts/values-prod.yaml --kubeconfig /etc/rancher/k3s/k3s.yaml'
            }
        }
    }
}
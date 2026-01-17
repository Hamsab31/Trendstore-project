pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "h4meed/trend-prod:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Hamsab31/Trendstore-project.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin
                      docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                  sed -i 's|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}|g' deployment.yaml
                  kubectl apply -f deployment.yaml
                  kubectl apply -f service.yaml
                """
            }
        }
    }
}
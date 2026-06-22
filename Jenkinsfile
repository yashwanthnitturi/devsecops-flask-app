pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/yashwanthnitturi/devsecops-flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devsecops-app:${BUILD_NUMBER} .'
            }
        }

        stage('Push To ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region ap-south-1 | docker login \
                --username AWS \
                --password-stdin 123185598576.dkr.ecr.ap-south-1.amazonaws.com

                docker tag devsecops-app:${BUILD_NUMBER} \
                123185598576.dkr.ecr.ap-south-1.amazonaws.com/devsecops-app:${BUILD_NUMBER}

                docker push \
                123185598576.dkr.ecr.ap-south-1.amazonaws.com/devsecops-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}

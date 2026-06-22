pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yashwanthnitturi/devsecops-flask-app.git'
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
                ...
                '''
            }
        }
    }
}

pipeline {
agent any

```
stages {

    stage('Checkout') {
        steps {
            git branch: 'main',
                url: 'https://github.com/yashwanthnitturi/devsecops-flask-app.git'
        }
    }

    stage('SonarQube Scan') {
        steps {
            script {
                def scannerHome = tool 'sonar-scanner'

                withSonarQubeEnv('SonarQube') {
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=devsecops-flask-app \
                    -Dsonar.projectName=devsecops-flask-app \
                    -Dsonar.sources=. \
                    -Dsonar.python.version=3.12
                    """
                }
            }
        }
    }

    stage('Trivy Filesystem Scan') {
        steps {
            sh 'trivy fs --severity HIGH,CRITICAL .'
        }
    }

    stage('Build Docker Image') {
        steps {
            sh 'docker build -t devsecops-app:${BUILD_NUMBER} .'
        }
    }

    stage('Trivy Image Scan') {
        steps {
            sh 'trivy image --severity HIGH,CRITICAL devsecops-app:${BUILD_NUMBER}'
        }
    }

    stage('Push To ECR') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'aws-ecr',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )]) {

                sh '''
                export AWS_DEFAULT_REGION=ap-south-1

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

    stage('Update GitOps Repo') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'github-token',
                usernameVariable: 'GIT_USER',
                passwordVariable: 'GIT_PASS'
            )]) {

                sh '''
                rm -rf gitops

                git clone https://${GIT_USER}:${GIT_PASS}@github.com/yashwanthnitturi/aws-eks-devsecops-gitops.git gitops

                sed -i "s|image: .*|image: 123185598576.dkr.ecr.ap-south-1.amazonaws.com/devsecops-app:${BUILD_NUMBER}|g" gitops/kubernetes/flask-app/deployment.yaml

                cd gitops

                git config user.email "jenkins@local"
                git config user.name "Jenkins"

                git add .
                git commit -m "Update image to build ${BUILD_NUMBER}" || true

                git push origin main
                '''
            }
        }
    }
}


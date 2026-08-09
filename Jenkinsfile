pipeline {
    agent any

    stages {
        stage('Install Dependencies') {
            steps {
                bat 'python -m pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'MONGO_URI',
                        variable: 'MONGO_URI'
                    )
                ]) {
                    bat 'python -m pytest'
                }
            }
        }
       stage('Docker Build') {
            steps {
                bat 'docker build -t student-registration-app:latest .'
            }
        }
      stage('ECR Login') {
           steps {
                withCredentials([
               [$class: 'AmazonWebServicesCredentialsBinding',
               credentialsId: 'aws-ecr']
        ]) {
            bat 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 251523190381.dkr.ecr.us-east-1.amazonaws.com'
        }
    }
}
    }
}
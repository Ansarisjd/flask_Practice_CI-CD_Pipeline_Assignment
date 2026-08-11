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

        stage('Docker Tag') {
            steps {
                bat 'docker tag student-registration-app:latest 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:latest'
            }
        }
       stage('Docker Push') {
            steps {
               bat 'docker push 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:latest'
    }
}

stage('Test EC2 SSH') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: '4a82c7e8-1eb2-430c-bbbc-f62e63d03635',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat '''
                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "echo EC2 SSH connection successful"
                    '''
                }
            }
        }

    }
}
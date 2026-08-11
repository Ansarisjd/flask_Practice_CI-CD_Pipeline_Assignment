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

stage('Deploy to EC2') {
    steps {
        withCredentials([
            sshUserPrivateKey(
                credentialsId: 'ec2-jenkins-ssh',
                keyFileVariable: 'SSH_KEY',
                usernameVariable: 'SSH_USER'
            )
        ]) {
            bat '''
                icacls "%SSH_KEY%" /inheritance:r
                icacls "%SSH_KEY%" /remove "BUILTIN\\Users"
                icacls "%SSH_KEY%" /grant:r "SYSTEM:F"

                ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 251523190381.dkr.ecr.us-east-1.amazonaws.com"

                ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "sudo docker pull 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:latest"

                ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "sudo docker stop student-registration-app || true"

                ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "sudo docker rm student-registration-app || true"

                ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@3.89.107.221 "sudo docker run -d --name student-registration-app --env-file /home/ubuntu/student-registration.env -p 5000:5000 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:latest"
            '''
        }
    }
}
    }
}
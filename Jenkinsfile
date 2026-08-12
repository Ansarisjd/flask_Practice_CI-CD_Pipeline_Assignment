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
                bat 'echo Git Commit SHA: %GIT_COMMIT%'
                bat 'docker build -t student-registration-app:%GIT_COMMIT% .'
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
                bat 'docker tag student-registration-app:%GIT_COMMIT% 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:%GIT_COMMIT%'
            }
        }

        stage('Docker Push') {
            steps {
                bat 'docker push 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:%GIT_COMMIT%'
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'ec2-jenkins-ssh',
                        variable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        icacls "%SSH_KEY%" /inheritance:r
                        icacls "%SSH_KEY%" /grant:r "SYSTEM:F"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 251523190381.dkr.ecr.us-east-1.amazonaws.com"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "sudo docker pull 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:%GIT_COMMIT%"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "sudo docker stop student-registration-app || true"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "sudo docker rm student-registration-app || true"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "sudo docker run -d --name student-registration-app --env-file /home/ubuntu/student-registration.env -p 5000:5000 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:%GIT_COMMIT%"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'ec2-jenkins-ssh',
                        variable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        icacls "%SSH_KEY%" /inheritance:r
                        icacls "%SSH_KEY%" /grant:r "SYSTEM:F"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@100.53.95.23 "curl -f http://localhost:5000/health"
                    '''
                }
            }
        }
    }

    post {

        success {
            mail(
                to: 'ansarisjdmohd3072@gmail.com',
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Jenkins Pipeline Successful

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Application deployed successfully to EC2.

Git Commit:
${env.GIT_COMMIT}

Build URL:
${env.BUILD_URL}
"""
            )
        }

        failure {
            mail(
                to: 'ansarisjdmohd3072@gmail.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Jenkins Pipeline Failed

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}

Please check the Jenkins console output.

Git Commit:
${env.GIT_COMMIT}

Build URL:
${env.BUILD_URL}
"""
            )
        }
    }
}
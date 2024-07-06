pipeline {
    agent any

    environment {
        AWS_ECR_REPOSITORY = '407916562905.dkr.ecr.us-east-1.amazonaws.com/hello-people'
        DOCKER_IMAGE = "${AWS_ECR_REPOSITORY}:v${env.BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
        EC2_USER = 'ec2-user'
        EC2_HOST = 'ec2-34-207-146-203.compute-1.amazonaws.com'
        EC2_KEY = '/home/ec2-user/jala-key.pem'
        MAIL_RECIPIENT= 'mdfaizanali7386@gmail.com'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Faizan-faizan1/webapp-pipeline.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ECR_REPOSITORY}
                        docker tag ${DOCKER_IMAGE} ${AWS_ECR_REPOSITORY}:${env.BUILD_NUMBER}
                        docker push ${AWS_ECR_REPOSITORY}:${env.BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                script {
                    sh '''
                        ssh -i ${EC2_KEY} ${EC2_USER}@${EC2_HOST} "
                        docker pull ${AWS_ECR_REPOSITORY}:${env.BUILD_NUMBER}
                        docker stop app || true
                        docker rm app || true
                        docker run -d --name app -p 80:80 ${AWS_ECR_REPOSITORY}:${env.BUILD_NUMBER}
                        "
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: "${MAIL_RECIPIENT}",
                 subject: "SUCCESS: Build #${env.BUILD_NUMBER}",
                 body: "The build ${env.BUILD_NUMBER} was successful."
        }
        failure {
            mail to: "${MAIL_RECIPIENT}",
                 subject: "FAILURE: Build #${env.BUILD_NUMBER}",
                 body: "The build ${env.BUILD_NUMBER} failed."
        }
    }
}

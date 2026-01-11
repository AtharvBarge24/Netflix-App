
pipeline {
    agent any

environment {
    AWS_ACCESS_KEY_ID     = credentials('aws-access-key')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    ECR_REPO = credentials('ecr-url')
    SONAR_AUTH_TOKEN = credentials('sonar-token')
    IMAGE_NAME = "netflix-app"
    TAG = "latest"
}

    stages {

        stage('Clone Repo') {
            steps {
                git branch: 'feature-atharv', url: 'https://github.com/AtharvBarge24/Netflix-App.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=netflix-app \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://sonarqube:9000 \
                      -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$TAG .
                docker tag $IMAGE_NAME:$TAG $ECR_REPO/$IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                aws configure set region us-west-1
                aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_REPO/$IMAGE_NAME:$TAG
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                docker system prune -af
                rm -rf *
                '''
            }
        }
    }
}


// pipeline {
//     agent any

//     environment {
//         scannerHome = tool 'SonarScanner'
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Sonar Scan') {
//             steps {
//                 withSonarQubeEnv('SonarQube') {
//                     sh '''
//                     $scannerHome/bin/sonar-scanner
//                     '''
//                 }
//             }
//         }
//     }
// }
pipeline {
    agent any
    stages {
        stage('Checkout branch') {
            steps {
                withFolderProperties {
                    checkout scm
                    sh "git checkout ${env.GH_BRANCH}"
                }
            }
        }
        stage('Prepare') {
            steps {
                withFolderProperties {
                    sh 'env'
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com"
                }
            }
        }
        stage('Build image') {
            steps {
                withFolderProperties {
                    script {
                        app = docker.build("${env.SERVICE_NAME}", "./service")
                    }
                }
            }
        }
        stage('Push image') {
            steps {
                withFolderProperties {
                    script {
                        docker.withRegistry("https://${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com") {
                            app.push("${env.BUILD_NUMBER}")
                            app.push("latest")
                        }
                    }
                }
            }
        }
    }
    post {
            always {
                withFolderProperties {
                    sh "docker rmi ${env.SERVICE_NAME}:${env.BUILD_NUMBER} || true"
                    sh "docker rmi ${env.SERVICE_NAME}:latest || true"
                    sh "docker rmi ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.SERVICE_NAME}:${env.BUILD_NUMBER} || true"
                    sh "docker rmi ${env.AWS_ACCOUNT_ID}.dkr.ecr.${env.AWS_REGION}.amazonaws.com/${env.SERVICE_NAME}:latest || true"
                    cleanWs()
                }
            }
    }
}

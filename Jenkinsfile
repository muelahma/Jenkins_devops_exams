pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-access-token')
        DOCKER_REPO = 'muelahma'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/muelahma/Jenkins_devops_exams.git', branch: 'main'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_REPO}/movie-service:latest ./movie-service'
                    sh 'docker build -t ${DOCKER_REPO}/cast-service:latest ./cast-service'
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    sh '''
                        echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                    '''
                    sh 'docker push ${DOCKER_REPO}/movie-service:latest'
                    sh 'docker push ${DOCKER_REPO}/cast-service:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Determine the namespace based on the branch
                    def branchName = env.BRANCH_NAME
                    def namespace = ''

                    if (branchName == 'dev') {
                        namespace = 'dev'
                    } else if (branchName == 'qa') {
                        namespace = 'qa'
                    } else if (branchName == 'staging') {
                        namespace = 'staging'
                    } else if (branchName == 'main') {
                        input message: 'Deploy to Production?', ok: 'Deploy'
                        namespace = 'prod'
                    } else {
                        namespace = 'dev'
                    }
                    sh """
                        helm upgrade --install movie-service ./helm/movie-service --namespace ${namespace} --set image.repository=${DOCKER_REPO}/movie-service,image.tag=latest
                        helm upgrade --install cast-service ./helm/cast-service --namespace ${namespace} --set image.repository=${DOCKER_REPO}/cast-service,image.tag=latest
                        helm upgrade --install nginx ./helm/nginx --namespace ${namespace}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed. Please check the logs.'
        }
    }
}
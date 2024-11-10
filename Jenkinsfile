pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-access-token')
        DOCKER_REPO = 'muelahma'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/muelahma/Jenkins_devops_exams.git', branch: 'master'
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
                        echo docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                    '''
                    sh 'docker push ${DOCKER_REPO}/movie-service:latest'
                    sh 'docker push ${DOCKER_REPO}/cast-service:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            environment {
                NAMESPACE = ''  // Define NAMESPACE as an environment variable
            }
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    if (branchName == 'dev') {
                        env.NAMESPACE = 'dev'
                    } else if (branchName == 'qa') {
                        env.NAMESPACE = 'qa'
                    } else if (branchName == 'staging') {
                        env.NAMESPACE = 'staging'
                    } else if (branchName == 'main') {
                        input message: 'Deploy to Production?', ok: 'Deploy'
                        env.NAMESPACE = 'prod'
                    } else {
                        env.NAMESPACE = 'dev'
                    }

                    // Print NAMESPACE for debugging
                    echo "Deploying to namespace: ${env.NAMESPACE}"

                    withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG')]) {
                        // Use NAMESPACE environment variable directly in the Helm commands
                        sh """
                            helm upgrade --install movie-service ./helm/movie-service --namespace ${env.NAMESPACE} --set image.repository=${DOCKER_REPO}/movie-service,image.tag=latest
                            helm upgrade --install cast-service ./helm/cast-service --namespace ${env.NAMESPACE} --set image.repository=${DOCKER_REPO}/cast-service,image.tag=latest
                            helm upgrade --install nginx ./helm/nginx --namespace ${env.NAMESPACE}
                        """
                    }
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

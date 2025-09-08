pipeline {
    agent any

    environment {
        DOCKER_IMAGE_DEV  = "khalidali07/nodejs-app:dev-${env.BUILD_NUMBER}"
        DOCKER_IMAGE_PROD = "khalidali07/nodejs-app:prod-${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/KhalidAli555/Nodejs-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE_DEV ."
                sh "docker tag $DOCKER_IMAGE_DEV $DOCKER_IMAGE_PROD"
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push $DOCKER_IMAGE_DEV"
                sh "docker push $DOCKER_IMAGE_PROD"
            }
        }

        stage('Deploy Database to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE
                    kubectl apply -f database1-mysql.yml -n database1
                    kubectl get pods -n database1
                    '''
                }
            }
        }

        stage('Deploy Node.js App to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE
                    kubectl apply -f dev1-deployment.yml -n dev1
                    kubectl get pods -n dev1
                    '''
                }
            }
        }

        stage('Approval for Prod Deployment') {
            steps {
                input message: "Approve deployment to PROD?"
            }
        }

        stage('Deploy Database to Prod') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE
                    kubectl apply -f database1-mysql.yml -n database1
                    kubectl get pods -n database1
                    '''
                }
            }
        }

        stage('Deploy Node.js App to Prod') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    export KUBECONFIG=$KUBECONFIG_FILE
                    kubectl apply -f prod1-deployment.yml -n prod1
                    kubectl get pods -n prod1
                    '''
                }
            }
        }

    }

    post {
        always {
            echo "✅ Full Pipeline completed"
        }
        cleanup {
            sh 'docker logout'
        }
    }
}

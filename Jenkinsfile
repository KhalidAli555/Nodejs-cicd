pipeline {
    agent any

    environment {
        DOCKER_IMAGE_DEV  = "khalidali07/nodejs-app:dev-${env.BUILD_NUMBER}"
        DOCKER_IMAGE_PROD = "khalidali07/nodejs-app:prod-${env.BUILD_NUMBER}"
        AZURE_VM_HOST     = "20.108.64.203"
        AZURE_VM_USER     = "IKSCluster"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "main", url: 'https://github.com/KhalidAli555/Nodejs-cicd.git'
            }
        }

        stage('Build & Push Docker on Azure VM') {
            steps {
                // Docker credentials ka block
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sshagent(['azure-vm-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no $AZURE_VM_USER@$AZURE_VM_HOST '
                            cd ~/workspace
                            git clone -b main https://github.com/KhalidAli555/Nodejs-cicd.git || cd Nodejs-cicd && git pull
                            cd Nodejs-cicd
                            docker build -t $DOCKER_IMAGE_DEV .
                            docker tag $DOCKER_IMAGE_DEV $DOCKER_IMAGE_PROD
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $DOCKER_IMAGE_DEV
                            docker push $DOCKER_IMAGE_PROD
                            docker logout
                        '
                        """
                    }
                }
            }
        }

        stage('Deploy Database to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=/var/jenkins_home/kubeconfig
                    kubectl apply -f database1-mysql.yml -n database1
                    kubectl get pods -n database1
                    """
                }
            }
        }

        stage('Deploy Node.js App to Dev') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=/var/jenkins_home/kubeconfig
                    kubectl apply -f dev1-deployment.yml -n dev1
                    kubectl get pods -n dev1
                    """
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
                    sh """
                    export KUBECONFIG=/var/jenkins_home/kubeconfig
                    kubectl apply -f database1-mysql.yml -n database1
                    kubectl get pods -n database1
                    """
                }
            }
        }

        stage('Deploy Node.js App to Prod') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=/var/jenkins_home/kubeconfig
                    kubectl apply -f prod1-deployment.yml -n prod1
                    kubectl get pods -n prod1
                    """
                }
            }
        }

    }

    post {
        always {
            echo "✅ Full Pipeline completed"
        }
    }
}

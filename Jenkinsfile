pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'   // your Jenkins DockerHub creds
        SSH_CREDENTIALS = 'devops-key'        // your Jenkins SSH private key for Ansible
        DOCKER_IMAGE = 'rohith252/apachewebsite'
        K8S_DEPLOYMENT = 'apache-deployment'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache with Ansible') {
            steps {
                sshagent (credentials: [SSH_CREDENTIALS]) {
                    sh '''
                        ansible-playbook -i ~/.ssh/apachewebsite/inventory.ini \
                        ~/.ssh/apachewebsite/install_apache.yml
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed -i "s|image: .*$|image: $DOCKER_IMAGE:${BUILD_NUMBER}|" deployment.yml
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                    kubectl rollout status deployment/$K8S_DEPLOYMENT --namespace=$K8S_NAMESPACE
                '''
            }
        }
    }
}

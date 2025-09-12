pipeline {
    agent any

    environment {
        REPO_PATH = '/var/lib/jenkins/.ssh/apachewebsite/apachewebsite'
        INVENTORY = '/var/lib/jenkins/.ssh/apachewebsite/inventory.ini'
        DOCKER_IMAGE = 'rohith252/apachewebsite:latest'
        DOCKERHUB_CREDENTIALS = 'docker-hub'
        ANSIBLE_CREDENTIALS = 'devops-node-key'
    }

    stages {

        stage('Checkout GitHub Repo') {
            steps {
                git branch: 'master', url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache via Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${ANSIBLE_CREDENTIALS}", keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    sh """
                        ansible-playbook -i ${INVENTORY} ${REPO_PATH}/install_apache.yml
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t ${DOCKER_IMAGE} ${REPO_PATH}
                    """
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f ${REPO_PATH}/deployment.yml
                    kubectl apply -f ${REPO_PATH}/service.yml
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    kubectl get pods
                    kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output for errors.'
        }
    }
}

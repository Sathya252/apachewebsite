pipeline {
    agent any

    environment {
        // DockerHub credentials ID in Jenkins
        DOCKER_CREDENTIALS = 'docker-hub'
        IMAGE_NAME = 'rohith252/apachewebsite:latest'
        ANSIBLE_INVENTORY = '/var/lib/jenkins/.ssh/apachewebsite/inventory.ini'
        ANSIBLE_PLAYBOOK = '/var/lib/jenkins/.ssh/apachewebsite/install_apache.yml'
        KUBE_DEPLOYMENT = '/var/lib/jenkins/.ssh/apachewebsite/apachewebsite/deployment.yml'
        KUBE_SERVICE = '/var/lib/jenkins/.ssh/apachewebsite/apachewebsite/service.yml'
    }

    stages {
        stage('Checkout GitHub') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache via Ansible') {
            steps {
                sh """
                    ansible-playbook -i ${ANSIBLE_INVENTORY} ${ANSIBLE_PLAYBOOK}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${KUBE_DEPLOYMENT}"
                sh "kubectl apply -f ${KUBE_SERVICE}"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods"
                sh "kubectl get svc"
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed! Check logs."
        }
    }
}

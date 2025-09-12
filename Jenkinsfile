pipeline {
    agent any

    environment {
        REPO_PATH = '/var/lib/jenkins/.ssh/apachewebsite/apachewebsite'
        DOCKER_IMAGE = 'rohith252/apachewebsite:latest'
        DOCKERHUB_CREDENTIALS = 'docker-hub' // Jenkins credential ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rohith252/apachewebsite.git'
            }
        }

        stage('Install Apache via Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: "${REPO_PATH}/install_apache.yml",
                    inventory: "${REPO_PATH}/inventory.ini"
                )
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
    }
}

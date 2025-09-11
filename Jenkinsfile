pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'rohith252/apachewebsite:latest'
    }

    stages {

        stage('Checkout GitHub') {
            steps {
                git branch: 'master', url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache via Ansible') {
            steps {
                ansiblePlaybook(
                    playbook: 'install_apache.yml',
                    inventory: 'inventory.ini',
                    credentialsId: 'devops-node-key',
                    sudoUser: 'devops'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

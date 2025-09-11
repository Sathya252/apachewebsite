pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')   // your DockerHub creds ID
        APP_NAME = "apachewebsite"
        IMAGE_NAME = "rohith252/apachewebsite"             // replace with your DockerHub repo
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache with Ansible') {
            steps {
                sshagent (credentials: ['devops-key']) {
                    sh '''
                        ansible-playbook -i ~/.ssh/apachewebsite/apachewebsite/inventory.ini \
                                         ~/.ssh/apachewebsite/apachewebsite/install_apache.yml
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                        docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                '''
            }
        }
    }
}

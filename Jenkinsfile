pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')   // your DockerHub creds
        DOCKER_IMAGE = "rohith252/apachewebsite"           // change if needed
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache with Ansible') {
            steps {
                sshagent (credentials: ['devops-key']) {
                    sh '''
                        ansible-playbook -i inventory.ini install_apache.yml
                    '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    # replace image in deployment.yml with new tag
                    sed -i "s|image:.*|image: $DOCKER_IMAGE:$BUILD_NUMBER|" deployment.yml
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                '''
            }
        }
    }
}

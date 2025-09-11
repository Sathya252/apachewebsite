pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')   // Jenkins DockerHub creds ID
        DOCKER_IMAGE = "rohith252/apachewebsite"           // change if needed
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed -i "s|image:.*|image: $DOCKER_IMAGE:$BUILD_NUMBER|" deployment.yml
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                '''
            }
        }
    }
}

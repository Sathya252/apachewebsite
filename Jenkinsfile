pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'  // disables strict host key checking
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'master', url: 'https://github.com/Sathya252/apachewebsite.git'
            }
        }

        stage('Install Apache with Ansible') {
            steps {
                sshagent(['devops-key']) {
                    sh '''
                        # ensure known_hosts has the node key
                        mkdir -p ~/.ssh
                        ssh-keyscan -H 172.31.89.150 >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts

                        # run ansible playbook
                        ansible-playbook -i inventory.ini install_apache.yml
                    '''
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh '''
                        docker build -t $DOCKER_USER/apache-website:latest .
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_USER/apache-website:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yml
                    kubectl apply -f service.yml
                    kubectl rollout status deployment/apache-deployment
                '''
            }
        }

    }
}

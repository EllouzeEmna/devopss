pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'emnaellouze123487'  // Replace with your Docker Hub username
        DOCKERHUB_PASSWORD = credentials('DockerHubPassword')  // Correct credential ID for Docker Hub password
        VM2_USER = 'recette'           // Replace with VM2 SSH user
        VM2_IP = '192.168.43.207'     // Replace with VM2 IP address
        VM2_APP_PATH = '~/app'        // Directory on VM2 to deploy
        DOCKER_CLI_EXPERIMENTAL = "enabled"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/EllouzeEmna/devopss.git', branch: 'main'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                docker-compose up -d
                docker run --rm --network=bank-automation_mynetwork bank-automation_backend mvn test
                docker-compose down
                '''
            }
        }


        stage('Push Docker Images to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'DockerHubPassword', variable: 'DockerHubPassword')]) {
                    sh 'docker login -u lightjarvis -p ${DockerHubPassword}'
                    sh 'docker push lightjarvis/angular-app:${DOCKER_TAG}'
                }
            }
        }


        stage('Deploy on VM2') {
            steps {
                sshagent(['ssh-credentials-ci']) {
                    sh '''
                    ssh $VM2_USER@$VM2_IP "mkdir -p $VM2_APP_PATH"
                    scp docker-compose.yml $VM2_USER@$VM2_IP:$VM2_APP_PATH/
                    ssh $VM2_USER@$VM2_IP "
                        cd $VM2_APP_PATH &&
                        docker-compose pull &&
                        docker-compose up -d
                    "
                    '''
                }
            }
        }
    }

    post {
        always {
                cleanWs()            
        }
    }
}

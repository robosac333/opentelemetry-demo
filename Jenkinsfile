pipeline {
    agent any

    environment {
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {


        // stage('Pre-pull Images with Auth') {
        // steps {
        //     script {
        //     docker.withRegistry('https://index.docker.io/v1/', 'jk-dh-tk') {
        //         sh 'docker pull valkey/valkey:8.1-alpine'
        //     }
        //     }
        // }
    
        stage('Build Images') {
            steps {
                script {
                    sh 'docker compose build'
                }
            }
        }

        stage('Start Services') {
            steps {
                script {
                    sh 'docker compose up -d'
                }
            }
        }

        stage('Wait for Services') {
            steps {
                // Optional: Wait for a few seconds to ensure services initialize
                sh 'sleep 10'
            }
        }

        stage('Verify Running Containers') {
            steps {
                script {
                    sh 'docker ps'
                }
            }
        }

        // Optional: Run tests or validations here
        // stage('Run Tests') {
        //     steps {
        //         sh 'docker compose exec <service_name> <test_command>'
        //     }
        // }

        stage('Tear Down') {
            steps {
                script {
                    sh 'docker compose down -v'
                }
            }
        }
    }
}

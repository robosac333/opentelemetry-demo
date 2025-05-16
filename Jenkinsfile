pipeline {
    agent {
        kubernetes {
            yamlFile 'kaniko-pod-template.yaml'
        }
    }
    environment {
        IMAGE = 'robosac333/opentelemetry-demo:latest'
    }
    stages {
        stage('Build and Push') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --dockerfile=Dockerfile \
                      --context=${WORKSPACE} \
                      --destination=docker.io/${IMAGE} \
                      --cleanup
                    '''
                }
            }
        }

        // stage('Build Images') {
        //     steps {
        //         script {
        //             sh 'docker compose build'
        //         }
        //     }
        // }

        // stage('Start Services') {
        //     steps {
        //         script {
        //             sh 'docker compose up -d'
        //         }
        //     }
        // }

        // stage('Wait for Services') {
        //     steps {
        //         // Optional: Wait for a few seconds to ensure services initialize
        //         sh 'sleep 10'
        //     }
        // }

        // stage('Verify Running Containers') {
        //     steps {
        //         script {
        //             sh 'docker ps'
        //         }
        //     }
        // }

        // Optional: Run tests or validations here
        // stage('Run Tests') {
        //     steps {
        //         sh 'docker compose exec <service_name> <test_command>'
        //     }
        // }

        // stage('Tear Down') {
        //     steps {
        //         script {
        //             sh 'docker compose down -v'
        //         }
        //     }
        // }
    }
}

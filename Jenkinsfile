pipeline {
    agent any
    stages {

        
        // stage('Pre-pull Images with Auth') {
        // steps {
        //     script {
        //     docker.withRegistry('https://index.docker.io/v1/', 'jk-dh-tk') {
        //         sh 'docker pull valkey/valkey:8.1-alpine'
        //     }
        //     }
        // }
    
//         stage('Build Images') {
//             steps {
//                 script {
//                     sh 'docker compose build'
//                 }
//             }
//         }

        stage('Push to ECR') {
            steps {
                script {
                    // Login to ECR
                    sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 242201279990.dkr.ecr.us-west-2.amazonaws.com'

                    // Build and tag the Docker image
                    sh 'docker compose build -t oteldemo/cicdpipeline .'

                    // Tag the image for ECR
                    sh 'docker tag oteldemo/cicdpipeline:latest 242201279990.dkr.ecr.us-west-2.amazonaws.com/oteldemo/cicdpipeline:latest'

                    // Push the image to ECR
                    sh 'docker push 242201279990.dkr.ecr.us-west-2.amazonaws.com/oteldemo/cicdpipeline:latest'
                }
            }
        }

//         stage('Start Services') {
//             steps {
//                 script {
//                     sh 'docker compose up -d'
//                 }
//             }
//         }
//
//         stage('Wait for Services') {
//             steps {
//                 // Optional: Wait for a few seconds to ensure services initialize
//                 sh 'sleep 10'
//             }
//         }
//
//         stage('Verify Running Containers') {
//             steps {
//                 script {
//                     sh 'docker ps'
//                 }
//             }
//         }

        // Optional: Run tests or validations here
        // stage('Run Tests') {
        //     steps {
        //         sh 'docker compose exec <service_name> <test_command>'
        //     }
        // }

//         stage('Tear Down') {
//             steps {
//                 script {
//                     sh 'docker compose down -v'
//                 }
//             }
//         }
    }
}

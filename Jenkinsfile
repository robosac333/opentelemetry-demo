pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        ECR_REGISTRY = '242201279990.dkr.ecr.us-west-2.amazonaws.com'
        IMAGE_NAME = 'oteldemo/cicdpipeline'
    }

    stages {
        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'sachin'
                ]]) {
                    script {
                        // Login to ECR
                        sh '''
                            aws ecr get-login-password --region $AWS_REGION | \
                            docker login --username AWS --password-stdin $ECR_REGISTRY
                        '''

                        // Build and tag Docker image
                        sh '''
                            docker compose build -t $IMAGE_NAME .
                            docker tag $IMAGE_NAME:latest $ECR_REGISTRY/$IMAGE_NAME:latest
                            docker push $ECR_REGISTRY/$IMAGE_NAME:latest
                        '''
                    }
                }
            }
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
//     }
// }

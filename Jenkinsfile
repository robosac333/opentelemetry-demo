pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        ECR_REGISTRY = '242201279990.dkr.ecr.us-west-2.amazonaws.com'
        IMAGE_NAME = 'oteldemo/cicdpipeline'
        SERVICE_NAME = 'cicdpipeline' 
        APP_NAME = 'otel-demo'
        ECR_REPOSITORY = "${ECR_REGISTRY}/${IMAGE_NAME}"
        K8S_NAMESPACE = 'webapps'
    }

    stages {
        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'sachin'
                ]]) {
                    script {
                        sh '''
                            echo "Logging into ECR..."
                            aws ecr get-login-password --region $AWS_REGION | \
                            docker login --username AWS --password-stdin $ECR_REGISTRY

                            echo "Building Docker image..."
                            docker compose build

                            # echo "Tagging image..."
                            # docker tag $IMAGE_NAME:latest-load-generator $ECR_REGISTRY/$IMAGE_NAME:latest-load-generator

                            echo "Tagging and pushing each image..."
                            for IMAGE in $(docker images --format "{{.Repository}}:{{.Tag}}" | grep oteldemo/cicdpipeline); do
                              NAME_TAG=$(echo $IMAGE | cut -d':' -f2)
                              docker tag $IMAGE $ECR_REGISTRY/oteldemo/cicdpipeline:$NAME_TAG
                              docker push $ECR_REGISTRY/oteldemo/cicdpipeline:$NAME_TAG
                            done

                            # echo "Pushing image to ECR..."
                            # docker push $ECR_REGISTRY/$IMAGE_NAME:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '', 
                    clusterName: 'opentelemetry-cluster', 
                    contextName: '', 
                    credentialsId: 'k8-token', 
                    namespace: "${env.K8S_NAMESPACE}", 
                    serverUrl: 'https://8D0F91A9A30A61724E5D09917F7D3EC4.gr7.us-west-2.eks.amazonaws.com'
                ]]) {
                    script {
sh '''
echo "Deploying each image from ECR to EKS..."

for TAG in $(docker images --format "{{.Tag}}" | grep -v '<none>'); do
DEPLOYMENT_NAME=$TAG
IMAGE_URI=${ECR_REGISTRY}/${IMAGE_NAME}:$TAG

echo "Applying deployment for ${DEPLOYMENT_NAME} with image ${IMAGE_URI}"

cat <<-EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
name: ${DEPLOYMENT_NAME}
namespace: ${K8S_NAMESPACE}
spec:
replicas: 1
selector:
    matchLabels:
    app: ${DEPLOYMENT_NAME}
template:
    metadata:
    labels:
        app: ${DEPLOYMENT_NAME}
    spec:
    containers:
    - name: ${DEPLOYMENT_NAME}
        image: ${IMAGE_URI}
        ports:
        - containerPort: 80
EOF

done
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

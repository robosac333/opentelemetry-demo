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
                ]]){
                    script{
sh '''
echo "Deploying to Kubernetes..."
git clone https://github.com/robosac333/opentelemetry-demo.git
cd opentelemetry-demo/kubernetes
kubectl apply -f opentelemetry-demo.yaml -n otel-demo
'''
                    }
                }

            }
        }
    }

}

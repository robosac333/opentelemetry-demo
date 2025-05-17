pipeline {
    agent any

    environment {
        AWS_REGION = 'us-west-2'
        ECR_REGISTRY = '242201279990.dkr.ecr.us-west-2.amazonaws.com'
        IMAGE_NAME = 'oteldemo/cicdpipeline'
        SERVICE_NAME = 'cicdpipeline' 
        APP_NAME = 'otel-demo'
        ECR_REPOSITORY = "${ECR_REGISTRY}/${IMAGE_NAME}"
        K8S_NAMESPACE = 'otel-demo'
        K8S_DEPLOYMENT = 'otel-demo'
        EXCLUDED_DEPLOYMENTS = "frontend-proxy grafana"
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
                    serverUrl: 'https://43B7F7363C5652FE933434304BA92ED8.gr7.us-west-2.eks.amazonaws.com'
                ]]){
                    script{
sh '''
echo "Deploying to Kubernetes..."
cd kubernetes
kubectl apply -f opentelemetry-demo.yaml -n $K8S_DEPLOYMENT --validate=false

DEPLOYMENTS=$(kubectl get deployments -n $K8S_NAMESPACE -o jsonpath='{.items[*].metadata.name}')

FAILED_DEPLOYMENTS=0
for DEPLOYMENT in $DEPLOYMENTS; do
    if echo "$EXCLUDED_DEPLOYMENTS" | grep -w "$DEPLOYMENT" > /dev/null; then
        echo "Skipping status check for excluded deployment: $DEPLOYMENT"
        continue
    fi
    echo "Checking deployment status for: $DEPLOYMENT"
    kubectl rollout status deployment/$DEPLOYMENT -n $K8S_NAMESPACE
    ROLLOUT_STATUS=$?

    if [ $ROLLOUT_STATUS -ne 0 ]; then
        echo "Rollout failed for $DEPLOYMENT. Rolling back..."
        kubectl rollout undo deployment/$DEPLOYMENT -n $K8S_NAMESPACE
        echo "Rollback complete for $DEPLOYMENT."
        FAILED_DEPLOYMENTS=$((FAILED_DEPLOYMENTS+1))
    else
        echo "Rollout successful for $DEPLOYMENT."
    fi
done

if [ $FAILED_DEPLOYMENTS -gt 0 ]; then
    echo "$FAILED_DEPLOYMENTS deployment(s) failed and were rolled back."
    exit 1
else
    echo "All deployments are ready and running successfully!"
fi
'''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! All services deployed."
        }
        failure {
            echo "Pipeline failed. Some deployments may have been rolled back. Check the logs for details."
        }
    }
}
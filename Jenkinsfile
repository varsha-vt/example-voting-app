pipeline{
    agent {label 'worker'}
    environment{
        AWS_DEFAULT_REGION = 'us-east-1'
        SERVICE_NAME = 'vote'
        TASK_FAMILY = 'vote-fargate-v1'
        ECS_CLUSTER = 'vote-application'
    }
    stages{
        stage('Git Checkout') {
            steps{
                checkout scm
            }
        }
        stage('Build Docker Image'){
            parallel{
                stage('Build Image'){
                    steps{
                        sh 'echo "Building the docker image"'
                        sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 270335494562.dkr.ecr.us-east-1.amazonaws.com'
                        sh 'cd vote && sudo docker build -t 270335494562.dkr.ecr.us-east-1.amazonaws.com/demovt:v${BUILD_NUMBER} .'
                        sh 'echo "Pushing the docker image"'
                        sh 'docker push 270335494562.dkr.ecr.us-east-1.amazonaws.com/demovt:v${BUILD_NUMBER}'
                    }
                }
                stage('Run UnitTest'){
                    steps{
                        echo "Running Unit Tests"
                    }
                }
            }
        }
        stage('Deploy in ECS') {
            steps{
                script{
                    sh'''
                    ECR_IMAGE="270335494562.dkr.ecr.us-east-1.amazonaws.com/demovt:v${BUILD_NUMBER}"
                    TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition "$TASK_FAMILY" --region "$AWS_DEFAULT_REGION")
                    NEW_TASK_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$ECR_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
                    NEW_TASK_INFO=$(aws ecs register-task-definition --region "$AWS_DEFAULT_REGION" --cli-input-json "$NEW_TASK_DEFINTIION")
                    NEW_REVISION=$(echo $NEW_TASK_INFO | jq '.taskDefinition.revision')
                    aws ecs update-service --cluster ${ECS_CLUSTER} \
                    --service ${SERVICE_NAME} \
                    --task-definition ${TASK_FAMILY}:${NEW_REVISION}'''
                }
            }
        }
    }
}

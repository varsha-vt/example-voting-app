pipeline{
    agent {label 'worker'}
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
                        sh 'cd vote'
                        sh 'docker build -t 270335494562.dkr.ecr.us-east-1.amazonaws.com/demovt:v${BUILD_NUMBER} .'
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
    }
}

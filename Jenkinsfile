pipeline {
    agent any

    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        AWS_DEFAULT_REGION='ap-southeast-1'
        AWS_ECS_CLUSTER="learn-jenkins-ecs-cluster-prod-attempt2"
        AWS_ECS_SERVICE="learn-jenkins-ecs-service-prod"
        AWS_ECS_TD="learn-jenkins-ecs-task-definition-prod"
    }

    stages {
        stage('Build Docker Image'){
            steps {
                sh 'docker build -t my-jenkins-app .'
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials(
                    [usernamePassword(
                        credentialsId: 'my-aws',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY',
                        usernameVariable: 'AWS_ACCESS_KEY_ID'
                    )]
                ) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }
    }
}
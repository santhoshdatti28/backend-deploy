pipeline {
    agent { label 'AGENT-1' }

    environment { 
        PROJECT = 'expense'
        COMPONENT = 'backend' 
        DEPLOY_TO = "production"
        REGION = "us-east-1"
    }

    options {
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    parameters {
        string(name: 'version', description: 'Enter the application version')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick environment')
    }

    stages {

        stage('Setup Environment') {
            steps {
                script {
                    appVersion = params.version
                    deployEnvironment = params.deploy_to
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withAWS(region: 'us-east-1', credentials: "aws-creds") {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name expense-${deployEnvironment}
                            kubectl get nodes
                            cd helm
                            sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${deployEnvironment}.yaml
                            helm upgrade --install $COMPONENT -n $PROJECT -f values-${deployEnvironment}.yaml . --force
                        """
                    }
                }
            }
        }

    }

    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }
        failure {
            echo 'I will run when pipeline is failed'
        }
        success {
            echo 'I will run when pipeline is success'
        }
    }
}

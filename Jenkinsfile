pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select the Terraform environment'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform_project') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Workspace') {
            steps {
                dir('terraform_project') {
                    sh '''
                        terraform workspace select "$ENVIRONMENT" ||
                        terraform workspace new "$ENVIRONMENT"
                    '''
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform_project') {
                    sh '''
                        terraform plan \
                          -var-file="$ENVIRONMENT/$ENVIRONMENT.tfvars" \
                          -out=tfplan
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input(
                    message: 'Review the Terraform plan. Do you want to apply it?',
                    ok: 'Apply'
                )
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform_project') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
    }

    post {
    success {
        ech 'Terraform deployment completed successfully.'
    }

    failure {
        echo 'Terraform deployment failed.'

        emailext(
            to: 'YOUR_EMAIL@example.com',
            subject: "[Jenkins] FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
Pipeline: ${env.JOB_NAME}
Build Number: #${env.BUILD_NUMBER}
Environment: ${env.ENVIRONMENT}
Status: ${currentBuild.currentResult}

Console Output:
${env.BUILD_URL}console

Build URL:
${env.BUILD_URL}

The Jenkins console log is attached to this email.
            """,
            attachLog: true
        )
    }

    aborted {
        echo 'Terraform deployment was aborted.'
    }
}
}

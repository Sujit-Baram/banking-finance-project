pipeline {
    agent any

    environment {
        TF_IN_AUTOMATION = "true"
    }

    stages {

        stage('Checkout & Build') {
            steps {
                deleteDir()
                git branch: 'main', url: 'https://github.com/Sujit-Baram/banking-finance-project.git'
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t sujitbaram/finance-tg:v1 .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    passwordVariable: 'PASS',
                    usernameVariable: 'USER'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push sujitbaram/finance-tg:v1
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh 'terraform init -no-color'
                }
            }
        }

        stage('Terraform Plan – PROD') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                    terraform workspace select prod || terraform workspace new prod
                    terraform plan -no-color
                    '''
                }
            }
        }

        stage('Terraform Apply – PROD (Manual Approval)') {
            steps {
                input message: 'Deploy infrastructure to PRODUCTION?', ok: 'Deploy'

                withCredentials([
                    string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh 'terraform apply -auto-approve -no-color'
                }
            }
        }
    }
}
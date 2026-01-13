pipeline {
    agent any
    stages{
        stage('build project'){
            steps{
                git branch: 'main', url: 'https://github.com/Sujit-Baram/banking-finance-project.git'
                sh 'mvn clean package'
              
            }
        }
        stage('Building  docker image'){
            steps{
                script{
                    sh 'docker build -t sujitbaram/finance-tg:v1 .'
                    sh 'docker images'
                }
            }
        }
        stage('push to docker-hub'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push sujitbaram/finance-tg:v1'
                }
            }
        }
        
        stage('Terraform Operations for test workspace') {
            steps {
                withCredentials([string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh '''
                terraform workspace select test || terraform workspace new test
                terraform init -no-color
                terraform plan -no-color
                terraform destroy -auto-approve -no-color
                '''
                }
            }
        }
       stage('Terraform destroy & apply for test workspace') {
            steps {
                withCredentials([string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh 'terraform apply -auto-approve -no-color'
                }
            }
       }
       stage('Terraform Operations for Production workspace') {
            when {
                expression { return currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                withCredentials([string(credentialsId: 'aws_access_key_id', variable: 'AWS_ACCESS_KEY_ID'), string(credentialsId: 'aws_secret_access_key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                sh '''
                terraform workspace select prod || terraform workspace new prod
                terraform init
                if terraform state show aws_key_pair.example 2>/dev/null; then
                    echo "Key pair already exists in the prod workspace"
                else
                    terraform import aws_key_pair.example key02 || echo "Key pair already imported"
                fi
                terraform destroy -auto-approve -no-color
                terraform apply -auto-approve -no-color
                '''
                }
            }
       }
    }
}

pipeline {
    agent any

    environment {
        ACR_NAME = 'acr-aks-tf98872'
        AZURE_CREDENTIALS_ID = 'jenkins-pipeline-sp'
        ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
        IMAGE_NAME = 'webapiacrtfjenkinsdocker'
        IMAGE_TAG = 'latest'
        RESOURCE_GROUP = 'rg-aks-tf'
        AKS_CLUSTER = 'AKSClustermj'
        TF_WORKING_DIR = '.'
        TERRAFORM_PATH = 'E:\\something\\Capgemini\\Cap-Training\\terraform.exe'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mahimajain01/aks-tf-jenkins.git'
            }
        }

        stage('Build .NET App') {
            steps {
                bat 'dotnet publish webApi-ask-tf/webApi-ask-tf.csproj -c Release -o out'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG% -f webApi-ask-tf/Dockerfile webApi-ask-tf"
            }
        }

       stage('Terraform Init') {
            steps {
                withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
                    bat """
                        echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
                        cd %TF_WORKING_DIR%
                        echo "Initializing Terraform..."
                        \"%TERRAFORM_PATH%\" init
                        """
                    }
                }
            }


       stage('Terraform Plan') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat """
            echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
            cd %TF_WORKING_DIR%
            "E:\\something\\Capgemini\\Cap-Training\\terraform.exe" plan -out=tfplan
            """
        }
    }
}



        stage('Terraform Apply') {
    steps {
        withCredentials([azureServicePrincipal(credentialsId: AZURE_CREDENTIALS_ID)]) {
            bat """
            echo "Navigating to Terraform Directory: %TF_WORKING_DIR%"
            cd %TF_WORKING_DIR%
            echo "Applying Terraform Plan..."
            terraform apply -auto-approve tfplan
            """
        }
    }
}
        stage('Login to ACR') {
            steps {
                bat "az acr login --name %ACR_NAME%"
            }
        }

        stage('Push Docker Image to ACR') {
            steps {
                bat "docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Get AKS Credentials') {
            steps {
                bat "az aks get-credentials --resource-group %RESOURCE_GROUP% --name %AKS_CLUSTER% --overwrite-existing"
            }
        }

        stage('Deploy to AKS') {
            steps {
                bat "kubectl apply -fwebApi-ask-tf/deployment.yaml"
            }
        }
    }

    post {
        success {
            echo 'All stages completed successfully!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}

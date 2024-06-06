pipeline {
    agent any
    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch to build from')
        string(name: 'AZURE_RESOURCE_GROUP', defaultValue: 'parthu-rg', description: 'Azure Resource Group')
        string(name: 'DATAFACTORY_NAME', defaultValue: 'parth-adf-testing', description: 'Data Factory Name')
     }
    environment {
        AZURE_TENANT_ID = 'dc304815-8478-486c-9d0e-ea5437aba38f'
        AZURE_SUBSCRIPTION_ID = '2e365525-e37e-43da-b869-8a4cc427db0b'
        AZURE_RESOURCE_GROUP = "adf-demo-azure-${ENVIRONMENT}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "main}",
                url: 'https://github.com/thkchva/testing-adf.git'
            }
        }
    
        stage('Create Artifacts'){
            steps {
                script {
                    nodejs(nodeJSInstallationName: 'npmnode') {
                        dir("build") {
                            sh "pwd"
                            sh 'npm install'
                            sh """
                                npm run build export ${{github.workspace}}/testing-adf/build  /subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/$AZURE_RESOURCE_GROUP/providers/Microsoft.DataFactory/factories/$DATAFACTORY_NAME "armDeploymentArtifact"
                            """
                        }
                    }
                }
            }
        }
        
        stage('Azure Login') {
            steps {
                withCredentials([azureServicePrincipal('92a43c0c-fb36-4806-a323-762d33b14aea')]) {
                    sh '''
                        AZURE_CONFIG_DIR=${{github.workspace}}/testing-adf/build
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID 
                        az account set -s $AZURE_SUBSCRIPTION_ID
                    '''
                }
            }
        }

        stage('Deploy ADF') {
            steps {
                script {
                    dir("build") {
                        sh '''
                            az deployment group create -g $AZURE_RESOURCE_GROUP --template-file armDeploymentArtifact/ARMTemplateForFactory.json --parameters factoryName=parth-adf-testing 
                    }
                }
            }
        }
    }
    post { 
        always { 
            sh 'az logout'
            cleanWs(cleanWhenNotBuilt: false,
                deleteDirs: true,
                disableDeferredWipeout: true,
                notFailBuild: true
            )
        }
    }
}

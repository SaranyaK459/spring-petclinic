pipeline {
    agent any

    environment {
        // Azure details based on your provided information
        AZURE_SUBSCRIPTION_ID = "d82fdb36-e398-48b3-b36b-d3520373269e"  // Your Azure subscription ID
        AZURE_RESOURCE_GROUP = "demo11"
        AZURE_WEBAPP_NAME = "saranya148715"
        AZURE_APP_SERVICE_PLAN = "ASP-demo11-b819"

        // Jenkins credential ID for Azure service principal
        AZURE_CREDENTIALS_ID = "azure-sp"  // Make sure this matches your Jenkins credential ID

        // Maven settings (optional)
        MAVEN_CMD = "mvn"

        // Artifact details
        ARTIFACT_PATH = "target/*.jar"
    }
    
    stages {
        // --- Continuous Integration ---

        stage('Checkout from Git') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh "${env.MAVEN_CMD} validate"
            }
        }

        stage('Package App') {
            steps {
                sh "${env.MAVEN_CMD} package"
            }
        }

        stage('Publish Artifact') {
            steps {
                archiveArtifacts artifacts: env.ARTIFACT_PATH, fingerprint: true
            }
        }

        // --- Continuous Deployment ---

        stage('Download Artifact') {
            steps {
                // Assuming this runs on the same agent, restore artifacts if needed
                // If deploying from different workspace, use copyArtifacts plugin or pipeline artifacts
                echo "Artifact assumed to be available in workspace"
            }
        }

        stage('Login to Azure') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.AZURE_CREDENTIALS_ID,
                    usernameVariable: 'AZURE_CLIENT_ID',
                    passwordVariable: 'AZURE_CLIENT_SECRET'
                )]) {
                    // You might want to inject tenant ID if needed, set as environment variable or here directly
                    sh '''
                        # Login to Azure using service principal
                        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET --tenant $AZURE_TENANT_ID
                        az account set --subscription $AZURE_SUBSCRIPTION_ID
                    '''
                }
            }
        }

        stage('Deploy to Azure Web App') {
            steps {
                sh """
                    # Deploy packaged .jar to Azure Web App (Java SE example)
                    az webapp deploy --resource-group ${env.AZURE_RESOURCE_GROUP} \\
                                     --name ${env.AZURE_WEBAPP_NAME} \\
                                     --type jar \\
                                     --src-path ${env.ARTIFACT_PATH}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

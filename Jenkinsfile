pipeline {
    agent any

    tools {
        maven 'Maven 3.6.3' // Ensure this matches the Maven installation name in Jenkins
    }

    environment {
        ImageName = 'my-app-image'
        BUILD_TAG = "latest"
    }

    stages {
        stage('Checkout From Git') {
            steps {
                git branch: 'main', url: 'https://github.com/SaranyaK459/spring-petclinic.git'
            }
        }

        stage('Maven Validate') {
            steps {
                echo 'Validating the project...'
                sh 'mvn validate'
            }
        }

        stage('Maven Compile') {
            steps {
                echo 'Compiling the project...'
                sh 'mvn compile'
            }
        }

        stage('Maven Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }

        stage('Maven Package') {
            steps {
                echo 'Packaging the project...'
                sh 'mvn package'
            }
        }

        stage('SonarCloud Analysis') {
            environment {
                SCANNER_HOME = tool 'sonar-scanner' // Matches tool config in Jenkins
            }
             steps {
                withCredentials([string(credentialsId: 'securetoken', variable: 'SONAR_TOKEN')]) {
                sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.organization=organisation1412 \
                -Dsonar.projectName=Jenkinsproject \
                -Dsonar.projectKey=organisation1412_jenkinsproject \
                -Dsonar.sources=src \
                -Dsonar.java.binaries=target/classes \
                -Dsonar.host.url=https://sonarcloud.io \
                -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Publish Sonar Report') {
            steps {
                echo 'Publishing SonarCloud report...'
                withCredentials([string(credentialsId: 'securetoken', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=organisation1412_jenkinsproject \
                        -Dsonar.organization=organisation1412 \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.token=$SONAR_TOKEN \
                        -Dsonar.qualitygate.wait=false
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t ${ImageName}:${BUILD_TAG} .
                    docker tag ${ImageName}:${BUILD_TAG} testregistry1311.azurecr.io/${ImageName}:${BUILD_TAG}
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Running Trivy scan...'
                sh '''
                    trivy image --format table --severity HIGH,CRITICAL \
                        --output trivy-report.txt testregistry1311.azurecr.io/${ImageName}:${BUILD_TAG}
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }

        stage('Login to ACR and Push Image') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZURE_USERNAME', passwordVariable: 'AZURE_PASSWORD'),
                    string(credentialsId: 'azure-tenant', variable: 'TENANT_ID')
                ]) {
                    script {
                        echo "Logging into Azure Container Registry..."
                        sh '''
                            az login --service-principal -u "$AZURE_USERNAME" -p "$AZURE_PASSWORD" --tenant "$TENANT_ID"
                            az acr login --name testregistry1311
                            docker push testregistry1311.azurecr.io/${ImageName}:${BUILD_TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    sh '''
                        az aks get-credentials --resource-group demo11 --name demo-aks
                        kubectl apply -f k8s/petclinic.yml
                        kubectl get all
                    '''
                }
            }
        }
    }
}

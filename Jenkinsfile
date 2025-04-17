pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'Maven'
    }

    environment {
        SCANNER_HOME = tool 'Sonar_scanner'
        ACR_NAME     = "rcr1983"
        IMAGE_NAME   = "mavenapp"
        TAG          = "v1"
        AZURE_CLIENT_ID     = credentials('azure-sp').username
        AZURE_CLIENT_SECRET = credentials('azure-sp').password
        AZURE_TENANT_ID = credentials('Az_tennantid') // Add this as a secret text in Jenkins
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'Gitlogin', url: 'https://github.com/sirisha-k83/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar_server') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar_server'
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Tag for ACR") {
            steps {
                script {
                    env.ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"
                    sh """
                        docker build -t ${IMAGE_NAME}:${TAG} .
                        docker tag ${IMAGE_NAME}:${TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }

        stage("Login to Azure & Push to ACR") {
            environment {
                AZURE_CLIENT_ID     = credentials('azure-sp').username
                AZURE_CLIENT_SECRET = credentials('azure-sp').password
            }
            steps {
                sh '''
                    az login --service-principal \
                        -u $AZURE_CLIENT_ID \
                        -p $AZURE_CLIENT_SECRET \
                        --tenant $AZURE_TENANT_ID

                    az acr login --name $ACR_NAME
                    docker push $ACR_LOGIN_SERVER/$IMAGE_NAME:$TAG
                '''
            }
        }
    }

    post {
        success {
            emailext(
                to: "mailtvs16@gmail.com",
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - SUCCESS",
                body: "✅ Build #${env.BUILD_NUMBER} was successful."
            )
        }

        failure {
            emailext(
                to: "mailtvs16@gmail.com",
                subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - FAILED",
                body: "❌ Build #${env.BUILD_NUMBER} failed. Please check Jenkins logs."
            )
        }
    }
}



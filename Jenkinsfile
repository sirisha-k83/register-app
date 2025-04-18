pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'Maven'
    }

    environment {
        SCANNER_HOME = tool 'Sonar_scanner'
        ACR_NAME     = "rcr1983"          // Your ACR name
        IMAGE_NAME   = "mavenapp"
        TAG          = "v1"
        RESOURCE_GROUP = 'rg1'
        CLUSTER_NAME   = 'myAKSCluster'
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
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Build Docker Image") {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage("Login & Push to ACR") {
            steps {
                script {
                    withCredentials([
                        usernamePassword(credentialsId: 'azure-sp', usernameVariable: 'AZURE_CLIENT_ID', passwordVariable: 'AZURE_CLIENT_SECRET'),
                        string(credentialsId: 'Az_tennantid', variable: 'AZURE_TENANT_ID')
                    ]) {
                        env.ACR_LOGIN_SERVER = "${ACR_NAME}.azurecr.io"

                        sh '''
                            az login --service-principal \
                                -u $AZURE_CLIENT_ID \
                                -p $AZURE_CLIENT_SECRET \
                                --tenant $AZURE_TENANT_ID

                            az acr login --name $ACR_NAME

                            docker tag ${IMAGE_NAME}:${TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                            docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                        '''
                    }
                }
            }
        }

        stage("Create AKS Cluster") {
            steps {
                sh '''
                    az aks create \
                        --resource-group $RESOURCE_GROUP \
                        --name $CLUSTER_NAME \
                        --node-count 2 \
                        --enable-addons monitoring \
                        --generate-ssh-keys \
                        --attach-acr $ACR_NAME
                '''
            }
        }

        stage("Get kubeconfig") {
            steps {
                sh '''
                    az aks get-credentials \
                        --resource-group $RESOURCE_GROUP \
                        --name $CLUSTER_NAME \
                        --overwrite-existing
                '''
            }
        }

        stage("Check Nodes") {
            steps {
                sh 'kubectl get nodes'
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

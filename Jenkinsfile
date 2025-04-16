 pipeline {
    agent any
    tools {
        jdk 'JDK'
        maven 'Maven'
    }
    environment {
        SCANNER_HOME = tool 'Sonar_scanner'
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

        stage("TRIVY FS Scan") {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t mavenapp ."
                        sh "docker tag mavenapp sirishak83/mavenapp:latest"
                        sh "docker push sirishak83/mavenapp:latest"
                    }
                }
            }
        }
    }

    post {
    success {
        emailext(
            to: "mailtvs16@gmail.com",
            subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - SUCCESS",
            body: "Good news! Build #${env.BUILD_NUMBER} was successful."
        )
    }

    failure {
        emailext(
            to: "mailtvs16@gmail.com",
            subject: "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - FAILED",
            body: "Oh no! Build #${env.BUILD_NUMBER} failed. Check Jenkins for details."
        )
    }
  }
}




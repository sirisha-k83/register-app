pipeline {
    agent any
    tools {
        jdk 'JDK'
        maven 'Maven'
    }
    environment {
            SCANNER_HOME = tool 'Sonar_scanner'
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'Gitlogin', url: 'https://github.com/sirisha-k83/register-app.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }

       stage("SonarQube Analysis"){
           steps {
                   script {
                        withSonarQubeEnv(credentialsId: 'sonar_server') { 
                        sh "mvn sonar:sonar"                    }
                   }
           }
       }

       stage("Quality Gate"){
           steps {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar_server'
                 }
                           }
        stage('TRIVY FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"  // Results stored in a text file
            }
        }
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                        sh "docker build -t mavenapp ."
                        sh "docker tag amazonprime sirishak83/mavenapp:latest"
                        sh "docker push sirishak83/mavenapp:latest"
                    }
                }
            }
        }
}
}

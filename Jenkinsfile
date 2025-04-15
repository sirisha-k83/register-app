pipeline {
    agent any
    tools {
        jdk 'JDK'
        maven 'Maven'
    }
    environment {
            SCANNER_HOME = tool 'sonar_server'
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
}
}

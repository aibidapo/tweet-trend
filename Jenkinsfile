pipeline {
    agent {
        node {
            label 'maven'
        }
    }

environment {
    PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
}    

    stages {
        stage("Build") {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('SonarQube analysis') {
            environment{
                scannerHome = tool 'ai-cloudops-sonar-scanner'
            }
                steps{
                    withSonarQubeEnv('ai-cloudops-sonarqube-server') { 
                        sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }        
    }
}

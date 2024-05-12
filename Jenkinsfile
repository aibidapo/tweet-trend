def registry = 'https://aicloudops.jfrog.io'
// def imageName = 'aicloudops.jfrog.io/aicloudops/tweet-trend'

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
        // stage('SonarQube analysis') {
        //     environment{
        //         scannerHome = tool 'ai-cloudops-sonar-scanner'
        //     }
        //         steps{
        //             withSonarQubeEnv('ai-cloudops-sonarqube-server') { 
        //                 sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
      // }      
      
        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifactory-cred"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "jarstaging/(*)",
                                  "target": "libs-release-local/{1}",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        }      
}
}

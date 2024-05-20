def registry = 'https://aicloudops.jfrog.io'

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
                echo '<--------------- Build Started --------------->'
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo '<--------------- Build Completed --------------->'
            }
        }

        stage("Test"){
            steps {
                echo '<--------------- UnitTest Started --------------->'
                sh 'mvn surefire-report:report'
                echo '<--------------- UnitTest Completed --------------->'
            }
        }
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'ai-cloudops-sonar-scanner'
            }
            steps {
                script {
                    try {
                        withSonarQubeEnv('ai-cloudops-sonarqube-server') { 
                            sh "${scannerHome}/bin/sonar-scanner"
                        }
                    } catch (Exception e) {
                        echo "SonarQube analysis failed: ${e.message}"
                        currentBuild.result = 'SUCCESS' // Set the build result to SUCCESS even if SonarQube analysis fails
                    }
                }
            }
        }

        stage("Quality Gate"){
            steps {
                script {
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
            error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
        }
        }
            }
        }              
      
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifactory-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "ai-cloudops-libs-release-local/{1}",
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

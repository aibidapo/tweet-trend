pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Clone Ttrend Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/aibidapo/tweet-trend'
            }
        }
    }
}

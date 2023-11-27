pipeline {
    agent {
        node {
            label 'maven'
        }
    }

environment {
    PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
}

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy -DskipTests=true'
            }

        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar-scanner';
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    // If you have configured more than one global server connection, you can specify its name
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
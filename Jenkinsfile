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
                echo "-------------Build Started--------------------"
                sh 'mvn clean deploy -DskipTests=true'
                echo "-------------Build Completed--------------------"
            }
        }

        stage('Test') {
            steps {
                echo "-------------Unit Test Started--------------------"
                sh 'mvn surefire-report:report'
                echo "-------------Unit Test Completed--------------------"
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

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQulaityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
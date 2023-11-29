def registry = 'https://pavaninfinity.jfrog.io/'
def imageName = 'pavaninfinity.jfrog.io/infinity-docker-local/ttrend'
def version   = '2.1.2'
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

        /*stage('SonarQube analysis') {
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
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }*/

        stage("Publish to Jrog") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"artifactory" ,  credentialsId:"Jfrog-Cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "infinity-libs-release-local/{1}",
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

        stage("Build Docker Image") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName+":"+version)
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }
        
        stage ("Publish to Docker Repo"){
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'Jfrog-Cred') {
                        app.push()
                    }    
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        } 
    }
}
def registry = 'https://verint1.jfrog.io'
def version = '2.1.2'
def imageName = 'https://verint1.jfrog.io/verint-docker/ttrend'

pipeline {
    agent {
        node{
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
}    

    stages {
         stage('build') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }
         stage("test"){
            steps{
                echo "----------unit test started--------------"
                sh 'mvn surefire-report:report'
                echo "----------unit test completed------------"
            }
         }
           stage('SonarQube analysis') {
            environment{
                scannerHome = tool 'sonar-scaner';     
            }
            steps{
                withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
                sh "${scannerHome}/bin/sonar-scanner"
            }
            }
        }
    //         stage("Quality Gate"){
    //             steps{
    //                 script {
    //                     timeout(time: 1, unit: 'HOURS') {
    //                 def qg = waitForQualityGate()
    //                 if (qg.status != 'OK') {
    //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
    //             }
    //         }
    //     }
    //   }
    //  }

         stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-maven-jenkins"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "my-maven-repo-libs-release-local/{1}",
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


        stage('docker build') {
            steps{
                script {
                app = docker.build ( imageName+":"+version)
                }
            }
        }

        stage('Deploy push') {
            steps{
                script {
                docker.withRegistry( registry, jfrog-maven-jenkins ) {
                    app.push()
                }
                }
            }
            }    


    }
}
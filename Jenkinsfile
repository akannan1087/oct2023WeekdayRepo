pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                 checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '19781f72-8f6c-4a4c-8b2f-f94d45449c45', url: 'https://github.com/akannan1087/oct2023WeekdayRepo']]) 
            }
        }
        
        stage ("build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
        }
        
        stage ("code scan") {
            steps {
                withSonarQubeEnv("SonarQube") {
                    sh "mvn clean install sonar:sonar -f MyWebApp/pom.xml"

                }
            }
        }
        
        stage ("coverage") {
            steps {
                jacoco()
            }
        }
        
        stage ("Nexus upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '748f59c7-2efd-479a-8dd9-ca6f3a3b0ae8', groupId: 'com.gcp', nexusUrl: 'ec2-52-72-169-202.compute-1.amazonaws.com:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'            }
        }
        
        stage ("DEV deploy") {
            steps {
                    deploy adapters: [tomcat9(credentialsId: 'fea81fa5-317b-443a-8a40-045cb19875b9', path: '', url: 'http://ec2-18-212-1-228.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ("Slack notify") {
            steps {
                slackSend channel: 'oct-2023-weekday-batch', message: 'DEV Deployment was success, please start testing in DEV Tomcat'
            }
        }
        
    //this CD 
     stage ('DEV Approve')  {
         steps {
            echo "Taking approval from DEV Manager for QA Deployment"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve QA Deployment?', submitter: 'admin'
            }
      }
     }
     
    stage ("QA deploy") {
        steps{
                    deploy adapters: [tomcat9(credentialsId: 'fea81fa5-317b-443a-8a40-045cb19875b9', path: '', url: 'http://ec2-18-212-1-228.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
       }
    }

    stage ("QA-Notify") {
        steps {
        slackSend channel: 'oct-2023-weekday-batch,qa-testing-team', message: 'QA Deployment was done, please start functional testing in QA Env.'
      }
    }
    stage ('QA Approve')  {
        steps {
            echo "Taking approval from QA Manager for PROD Deployment"     
            timeout(time: 3, unit: 'DAYS') {
            input message: 'Do you approve PROD Deployment?', submitter: 'admin'
            }
        }
    }
     
    stage ("PROD deploy") {
        steps {
            deploy adapters: [tomcat9(credentialsId: 'fea81fa5-317b-443a-8a40-045cb19875b9', path: '', url: 'http://ec2-18-212-1-228.compute-1.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
        }
    }
    
    stage ("PO-Notify") {
        steps {
            slackSend channel: 'oct-2023-weekday-batch,qa-testing-team,product-owners-teams', message: 'PROD Deployment was done, please inform end customers..'
         }
        }

    }
}

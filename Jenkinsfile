pipeline {

  options {
    ansiColor('xterm')
  }
    tools{
        maven 'maven'
    }

    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -DskipTests=true clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
          //def scannerHome = tool 'SonarQubeScanner';
            steps {
                withSonarQubeEnv('sonarqube-server'){
                //sh "${scannerHome}/bin/sonar-scanner"
                sh "mvn sonar:sonar"
                }
            }
        }
        stage('Upload artifact') {
            steps {
                  nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '192.168.0.15:8081',
                  groupId: 'com.example',
//                  version: "Version${env.BUILD_ID}_${env.BUILD_TIMESTAMP}",
//                  version: "Version_${env.BUILD_ID}",
                  version: "Version_latest",
                  repository: 'DemoApiRepository',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                      [artifactId: 'demoPipeline',
                       classifier: '',
                       file: 'target/demo-pipeline.war',
                       type: 'war']
                  ]
               )
            }
        }

        stage('Download Artifact from Nexus') {
            steps {
                script {
                    def nexusUrl = 'http://192.168.0.15:8081/repository/DemoApiRepository/'
                    def artifactPath = 'com/example/demoPipeline/Version_latest/demo-pipeline.war'
                    def downloadLocation = 'demo-pipeline.war'

                    sh "curl -o ${downloadLocation} ${nexusUrl}${artifactPath}"
                }
            }
        }

      stage('Deploy to Tomcat') {
          steps {
              script {
                def tomcatContainer = 'tomcat-manager'
                docker.image('maven:3.6.3-jdk-8').inside("-v ${env.WORKSPACE}:/usr/src/app") {
                    sh "docker cp ${env.WORKSPACE}/target/demo-pipeline.war ${tomcatContainer}:/usr/local/tomcat/webapps/"
                }
                sh "docker restart ${tomcatContainer}"
              }
           }
       }

    }
}
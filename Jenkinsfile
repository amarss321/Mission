pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven'
    }
    
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/amarss321/Mission.git' 
                }
        }
        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Maven test skip') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('trivy scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-reports.html .'
            }
        }
        stage('Sonar Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Mission -Dsonar.projectName=Mission \
                -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Maven build'){
            steps{
            sh 'mvn  package -DskipTests=true'
            }
        }
        stage('Deploy Artifacts To Nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'maven', jdk: 'jdk17', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                sh 'mvn  deploy -DskipTests=true'
              }
            }
        }
        stage('Building Docker image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker build -t amarnathvenkatam/mission:latest .'
                }
                }
            }
        }
        stage('trivy scan Docker image '){
            steps{
                sh 'trivy image --format table -o trivy-image-reports.html amarnathvenkatam/mission:latest '
            }
        }
    }
}
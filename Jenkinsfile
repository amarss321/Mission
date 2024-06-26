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
        stage('Pusing Docker image to DockerHub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh 'docker push amarnathvenkatam/mission:latest'
                }
                }
            }
        }
        stage('deploy to k8s'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'devops', contextName: '', credentialsId: 'kube-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://599766FAA04347FAB9F3C4D1EB40D8DC.gr7.us-east-1.eks.amazonaws.com') {
                    sh ' kubectl apply -f ds.yml -n webapps'
                    sleep 60
                }
            }
        }
        stage('verify k8s resorces in webapps'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: 'devops', contextName: '', credentialsId: 'kube-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://599766FAA04347FAB9F3C4D1EB40D8DC.gr7.us-east-1.eks.amazonaws.com') {
                    sh ' kubectl get svc -n webapps'
                    sh ' kubectl get pods -n webapps'
                }
            }
        }
        

    }
}
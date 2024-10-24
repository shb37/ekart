pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/shb37/ekart.git'
            }
        }
         stage('maven compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('unit test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ekart \
                    -Dsonar.projectKey=ekart -Dsonar.java.binaries=. '''
                }
            }
        }
        // stage('OWASP FS SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./' , odcInstallation: 'dpc'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        
        stage('Maven Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('deploy to Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                 sh 'mvn deploy -DskipTests=true'
                   
               }
            }
        }
        stage('Docker Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker')  {
                      sh 'docker build -t shb337/ekart:latest -f docker/Dockerfile . '
                    }
                }
               
            }
        }
        stage('Trivy image scan') {
            steps {
                sh 'trivy image shb337/ekart:latest > trivyreport.txt'
            }
        }
        stage('Docker Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker')  {
                      sh 'docker push shb337/ekart:latest '
                    }
                }
               
            }
        }
        stage("Proceed to Deploy"){
            steps{
                 script{
                    input(message: "Ready to Deploy!! Are you sure to proceed?", ok: "Proceed")
                }
            }
        }
        stage("Deploy the app"){
            steps{
                 sh 'docker run -d -p 8070:8070 shb337/ekart:latest'
            }
        }
    }
}

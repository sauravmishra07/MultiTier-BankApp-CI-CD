pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sauravmishra07/MultiTier-BankApp-CI-CD.git'
            }
        }
        
        stage('Compile App') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Testing') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Trivy Fs Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=gcbank \
                    -Dsonar.projectName=gcbank \
                    -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish To NEXUS') {
            steps {
                withMaven(globalMavenSettingsConfig: 'smproject', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
    }
}
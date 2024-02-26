pipeline {
    agent {
        docker {
            image 'cimg/android:2024.01' 
        }
    }
    environment {
        NEXUS_URL = '10.16.33.232:8081'
        NEXUS_CREDENTIAL_ID = '3' 
    }
    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }                  
        stage('Test') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
            
        }
        stage('Build') {
            steps {
                //sh './gradlew assembleRelease'
                sh './gradlew build --scan'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh "./gradlew sonar"
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "10.16.33.232:8081",
                        groupId: 'ok',
                        version: '3',
                        repository: 'myrepo',
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: 'ok', classifier: '', file: './app/build/outputs/apk/release/app-release-unsigned.apk ', type: 'apk']
                        ]
                    )
                }
            }
        }
    }
    post {
         always {
            echo "Cleaning . . . "
            cleanWs()
        }
    }
}

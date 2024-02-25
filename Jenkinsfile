pipeline {
    agent any
    environment {
        NEXUS_URL = '10.16.33.232:8081' // Update with your Nexus URL
        NEXUS_CREDENTIAL_ID = '3' // Jenkins credentials ID for Nexus
    }
    stages {
        stage('Test') {
            agent {
                docker {
                    image 'cimg/android:2024.01' // or any image with Java and Android SDK installed
                }
            }
            steps {
                git 'https://github.com/omarnaawad/WeatherApp-Android'
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
            
        }
        stage('Build') {
            agent {
                docker {
                    image 'cimg/android:2024.01' // or any image with Java and Android SDK installed
                }
            }
            steps {
                git 'https://github.com/omarnaawad/WeatherApp-Android'
                sh 'chmod +x gradlew'
                sh './gradlew assembleRelease'
            }
        }
        stage('SonarQube analysis') {
            agent {
                docker {
                    image 'cimg/android:2024.01' // or any image with Java and Android SDK installed
                }
            }
            steps {
                withSonarQubeEnv('mysonar') {
                    sh 'chmod +x gradlew'
                    sh "./gradlew sonar"
                }
            }
        }
        stage('Deploy to Nexus') {
            agent {
                docker {
                    image 'cimg/android:2024.01' // or any image with Java and Android SDK installed
                }
            }
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "10.16.33.232:8081",
                        groupId: 'ok', // Update with your groupId
                        version: '3', // Update with your version
                        repository: 'myrepo', // Update with your Nexus repository
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: 'ok', classifier: '', file: './app/build/outputs/apk/release/app-release-unsigned.apk ', type: 'apk']
                        ]
                    )
                }
            }
        }
    }
}

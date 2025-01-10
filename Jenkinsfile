pipeline {
    options {
        // This is required if you want to clean before build
        skipDefaultCheckout(true)
    }
    agent 
    { label 'android'}
    /*{
        docker {
            label 'docker'
            image 'cimg/android:2024.01.1' //cimg/android:2024.01 ccitest/android:2024.01.1 mobiledevops/android-sdk-image:34.0.0-jdk17 gradle:8-jdk17
        }
    }*/
    environment {
        NEXUS_URL = '10.16.33.232:8081'
        NEXUS_CREDENTIAL_ID = '3' 
        ANDROID_HOME = '/home/omar/android-sdk'
    }
    stages {
        stage('Clean') {
            steps {
                deleteDir()
            }
        }
        stage('SCM') {
            steps {
                checkout scm
            }
        }
        stage('setup') {
            steps {
                sh 'chmod +x gradlew'
            }   
        }
        
        stage('Build') {
            steps {
                sh 'java -version'
                //sh './gradlew assemble'
                sh './gradlew clean'
                sh './gradlew assembleRelease'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew test'
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
                        groupId: 'test',
                        version: '9',
                        repository: 'myrepo',
                        credentialsId: "${NEXUS_CREDENTIAL_ID}",
                        artifacts: [
                            [artifactId: 'movies', classifier: '', file: './app/build/outputs/apk/release/app-release-unsigned.apk', type: 'apk']
                        ]
                    )
                }
            }
        }
        stage ('SLackSend') {
            steps {
                //slackUploadFile filePath: "*./app/build/outputs/apk/release/app-release-unsigned.apk", channels: '#jen', initialComment:  "HEY That is APK"
                script {
                    echo "Current workspace: ${pwd()}"
                    slackUploadFile(
                        tokenCredentialId: 'SlackApp',
                        channels: '#jen',
                        file: 'app/build/outputs/apk/release/app-release-unsigned.apk',
                        initialComment:  "HEY That is APK"
                    )
                }
            }
        }
        stage('Upload to Slack') {
            steps {
                script {
                    def slackChannel = '#jen'
                     // Securely store this token using Jenkins credentials
                    def filePath = 'app/build/outputs/apk/release/app-release-unsigned.apk'
                    withCredentials([string(credentialsId: 'SlackApp', variable: 'SLACK_TOKEN')]) {
                        sh """
                        curl -F file=@${filePath} \
                             -F channels=${slackChannel} \
                             -F token=\$SLACK_TOKEN \
                             https://slack.com/api/files.upload
                        """
                    }
                    // Use curl to upload the file to Slack
                }
            }
        }
    }
    /*post {
        success{
            slackSend color: "good", message: "Success"
        }
        always { 
            cleanWs()
        }
    }*/
}

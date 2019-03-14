pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
             steps {
                sh 'mvn test'
             }
             post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
             }
         }
        stage('CodeCheck'){
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        sonar-scanner -X -Dsonar.language=java \
                        -Dsonar.projectKey=my-app \
                        -Dsonar.projectName=my-app \
                        -Dsonar.projectVersion=V1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.java.binaries=target/ \
                        -Dsonar.exclusions=src/test/**
                       """
                }
            }
        }
        stage("QualityGate") {
            agent none
            steps {
                timeout(time: 1, unit: "HOURS") {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
    }
}

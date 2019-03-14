pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    environment {
        SONAR_HOME = "${tool 'sonarqube-scanner'}"
        PATH="${env.SONAR_HOME}/bin:${env.PATH}"
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
            agent none
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn clean package sonar:sonar'
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

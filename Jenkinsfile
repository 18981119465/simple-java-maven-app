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
            agent {
                docker 'newtmitch/sonar-scanner:3.2.0-alpine'
            }
            step {
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
            timeout(time: 1, unit: "HOURS") {       // 防止获取回调出现异常情况，设置超时时间
                def qg = waitForQualityGate()
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
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

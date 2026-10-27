pipeline {
    agent any
    tools {
        maven 'M2_HOME'
        jdk 'JDK21'
    }
    stages {
        stage('Checkout code') {
            steps {
                git branch: 'master', url: 'https://github.com/FatmaDaldoul/Country-service'
            }
        }
        stage('Compile, Test & JaCoCo Report') {
            steps {
                sh 'mvn clean test jacoco:report'
            }
            post {
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'MySonarQubeServer', credentialsId: 'sonarqubePWD') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=country-service -Dsonar.projectName=country-service"
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy -DskipTests'
            }
        }

        stage('Build Dockerfile') {
            steps {
                sh 'docker build . -t my-country-service:$BUILD_NUMBER'
                withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u fatmadaldoul -p ${dockerhubpwd}'
                }
                sh 'docker tag my-country-service:6 fatmadaldoul/my-country-service:6'
                sh 'docker push fatmadaldoul/my-country-service:6'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-file', serverUrl: '') {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                    }
                }
            }
        }
    }
}

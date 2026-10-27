pipeline {
    agent any
    tools {
        maven 'M2_HOME'
        jdk 'JDK21'
    }
    environment {
        IMAGE = "fatmadaldoul/my-country-service"
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
                    sh 'echo $dockerhubpwd | docker login -u fatmadaldoul --password-stdin'
                }
                sh 'docker tag my-country-service:$BUILD_NUMBER $IMAGE:$BUILD_NUMBER'
                sh 'docker push $IMAGE:$BUILD_NUMBER'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-file', serverUrl: '') {
                        sh 'sed -i "s|image:.*|image: $IMAGE:$BUILD_NUMBER|" deployment.yaml'
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl apply -f service.yaml'
                        sh 'kubectl get deployments,pods,services'
                    }
                }
            }
        }
    }
}

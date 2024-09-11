

pipeline {
    agent {
        label 'slave01'
    }

    environment {
        SCANNER_HOME = tool 'sonar'
        DOCKERHUB_CREDENTIALS = credentials('dockerhubtoken')
         //{sh 'echo $(cat /frontend/frontend_version.txt)'}
     //{sh 'echo $(cat /backend/backend_version.txt)'}
    }
    stages {
        stage('code clone') {
            steps {
                git branch: 'devops', url: 'https://github.com/sakula023/wanderlust.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv("sonar") {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=WANDERLUST"
                }
            }
        }
        stage('build docker image - frontend') {
            steps {
                script {
                    echo "${WORKSPACE}"
                    env.FRONTEND_VERSION = readFile "${WORKSPACE}/frontend/frontend_version.txt"
                    dir('frontend') {
                        sh "docker build -t sakula23/wanderlust_frontend:${env.FRONTEND_VERSION} ."
                    }  
                }
                
            }
        }
        stage('build docker image - backend') {
            steps {

                script {
                    echo "${WORKSPACE}"
                    env.BACKEND_VERSION = readFile "${WORKSPACE}/backend/backend_version.txt"
                    dir('backend') {
                        sh "docker build -t sakula23/wanderlust_backend:${env.BACKEND_VERSION} ."
                    }  
                }
            }
        }
        stage('Scan Docker Image - frontend') {
            steps {
                script {
                    // Run Trivy to scan the Docker image
                    def trivyOutput = sh(script: "trivy image sakula23/wanderlust_frontend:${env.FRONTEND_VERSION}", returnStdout: true).trim()

                    // Display Trivy scan results
                    println trivyOutput

                    // Check if vulnerabilities were found
                    if (trivyOutput.contains("Total: 0")) {
                        echo "No vulnerabilities found in the Docker image."
                    } else {
                        echo "Vulnerabilities found in the Docker image."
                        // You can take further actions here based on your requirements
                        // For example, failing the build if vulnerabilities are found
                        // error "Vulnerabilities found in the Docker image."
                    }
                }
            }
        }
        stage('Scan Docker Image - backend') {
            steps {
                script {
                    // Run Trivy to scan the Docker image
                    def trivyOutput = sh(script: "trivy image sakula23/wanderlust_backend:${env.BACKEND_VERSION}", returnStdout: true).trim()

                    // Display Trivy scan results
                    println trivyOutput

                    // Check if vulnerabilities were found
                    if (trivyOutput.contains("Total: 0")) {
                        echo "No vulnerabilities found in the Docker image."
                    } else {
                        echo "Vulnerabilities found in the Docker image."
                        // You can take further actions here based on your requirements
                        // For example, failing the build if vulnerabilities are found
                        // error "Vulnerabilities found in the Docker image."
                    }
                }
            }
        }
        stage("Docker Push- all images"){
            steps{
            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            sh "docker push sakula23/wanderlust_frontend:${env.FRONTEND_VERSION}"
            sh "docker push sakula23/wanderlust_backend:${env.BACKEND_VERSION}"
            }
        }
    }
}
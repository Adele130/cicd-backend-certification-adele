pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = "tonuser/tasklist-backend"
        SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Adele130/cicd-backend-certification-adele.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Unit Tests') {
            steps {
                bat 'npm run test:coverage'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'npx sonar-scanner -Dsonar.login=%SONAR_TOKEN%'
                }
            }
        }

        stage('Build') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% -t %DOCKER_IMAGE%:latest ."
            }
        }

        stage('Trivy Scan') {
            steps {
                bat "trivy image --exit-code 0 --severity HIGH,CRITICAL %DOCKER_IMAGE%:latest"
            }
        }

        stage('SBOM Generation') {
            steps {
                bat "trivy image --format spdx-json -o sbom-spdx.json %DOCKER_IMAGE%:latest"
            }
            post {
                always {
                    archiveArtifacts artifacts: 'sbom-spdx.json'
                }
            }
        }

        stage('Docker Push') {
            steps {
                bat "echo %DOCKERHUB_CREDENTIALS_PSW% | docker login -u %DOCKERHUB_CREDENTIALS_USR% --password-stdin"
                bat "docker push %DOCKER_IMAGE%:%BUILD_NUMBER%"
                bat "docker push %DOCKER_IMAGE%:latest"
            }
        }
    }
}
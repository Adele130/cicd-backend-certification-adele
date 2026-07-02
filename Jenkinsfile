pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials-adele')
        DOCKER_IMAGE = "adele1304/tasklist-backend"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Adele130/cicd-backend-certification-adele.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm run test:coverage'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server-1') {
                    sh 'npx sonar-scanner'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$BUILD_NUMBER -t $DOCKER_IMAGE:latest ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL $DOCKER_IMAGE:latest"
            }
        }

        stage('SBOM Generation') {
            steps {
                sh "trivy image --format spdx-json -o sbom-spdx.json $DOCKER_IMAGE:latest"
            }
            post {
                always {
                    archiveArtifacts artifacts: 'sbom-spdx.json'
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker push $DOCKER_IMAGE:$BUILD_NUMBER"
                sh "docker push $DOCKER_IMAGE:latest"
            }
        }
    }
}
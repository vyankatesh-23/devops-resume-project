pipeline {
    agent any

    environment {
        IMAGE = 'vyankatesh23/devops-resume-site'
        TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE}:${TAG} -t ${IMAGE}:latest ."
            }
        }

        stage('Test') {
            steps {
                sh "docker run -d --name test-site -p 8082:8080 ${IMAGE}:${TAG}"
                sh 'sleep 3'
                sh 'curl -f http://localhost:8082'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f test-site || true'
        }
    }
}

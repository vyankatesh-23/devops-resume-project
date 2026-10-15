pipeline {
    agent any

    environment {
        IMAGE  = 'vyankatesh23/devops-resume-site'
        TAG    = "${BUILD_NUMBER}"
        APP_IP = '172.31.13.239'
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

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh 'echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin'
                    sh "docker push ${IMAGE}:${TAG}"
                    sh "docker push ${IMAGE}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['app-server-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=yes ubuntu@${APP_IP} '
                          set -e
                          docker pull ${IMAGE}:${TAG}
                          docker rm -f resume-site || true
                          docker run -d --name resume-site --restart unless-stopped -p 80:8080 ${IMAGE}:${TAG}
                        '
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f test-site || true'
            sh 'docker logout || true'
        }
    }
}

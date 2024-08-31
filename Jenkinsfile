pipeline {
    agent any
    stages {
        stage('SAST with Semgrep') {
            steps {
                script {
                    sh '''
                    apk add python3
                    apk add --update pipx
                    pipx install semgrep; pipx ensurepath; source ~/.bashrc
                    /root/.local/bin/semgrep scan --config auto --json > semgrep-report.json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'semgrep-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('DAST with OWASP ZAP') {
            steps {
                script {
                 sh '''
                    apk add --no-cache openjdk11-jre-headless wget unzip
                    wget https://github.com/zaproxy/zaproxy/releases/download/w2024-08-27/ZAP_WEEKLY_D-2024-08-27.zip
                    unzip ZAP_WEEKLY_D-2024-08-27.zip -d zap
                    zap/ZAP_D-2024-08-27/zap.sh -cmd -quickurl https://s410-exam.cyber-ed.space:8084 -quickout zap.json
                '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap.json ', allowEmptyArchive: true
                }
            }
        }
        stage('Container Security with Trivy') {
            steps {
                agent {
                label 'dind'
            }
                script {
                    sh '''
                    docker run aquasec/trivy repo https://github.com/Toxich4/nettu-meet-exam -f json -o json > trivy.json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.json', allowEmptyArchive: true
                }
            }
        }
    }
    post {
        always {
            echo 'Security pipeline completed.'
        }
        success {
            echo 'Success!'
        }
        failure {
            echo 'Failure!'
        }
    }
}

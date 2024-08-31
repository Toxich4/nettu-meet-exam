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
                    docker.image('owasp/zap2docker-stable').inside {
                        sh '''
                            zap-baseline.py -t https://s410-exam.cyber-ed.space:8084 -r zap_report.html
                            zap-cli report -o zap_report.xml -f xml
                        '''
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
                }
            }
        }
        stage('Container Security with Trivy') {
            steps {
                script {
                    sh '''
                    docker run -v ./report:/report aquasec/trivy repo https://github.com/Toxich4/nettu-meet-exam -f json -o /report/trivy.json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '/report/trivy-report.json', allowEmptyArchive: true
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

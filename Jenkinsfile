pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git url: 'https://github.com/Toxich4/nettu-meet-exam.git', branch: 'main'
            }
        }
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
        stage('SCA with Dependency-Check') {
            steps {
                script {
                    sh '''
                        apk add openjdk11
                        apk add curl
                        curl -Ls "https://github.com/jeremylong/DependencyCheck/releases/download/v10.0.3/dependency-check-10.0.3-release.zip" --output dependency-check.zip
                        unzip dependency-check.zip
                        ./dependency-check/bin/dependency-check.sh \
                            --project nettu-meet-exam\
                            --out dependency-check-report.html \
                            --scan . \
                            --nvdApiKey e2d1f143-c783-4be6-a928-22d3a9ad7fce
                    '''                
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dependency-check-report.html', allowEmptyArchive: true
                }
            }
        }

        stage('DAST with OWASP ZAP') {
            steps {
                script {
                 sh '''
                    apk add --no-cache openjdk11-jre-headless wget unzip
                    wget https://github.com/zaproxy/zaproxy/releases/download/v2.15.0/ZAP_2.15.0_Linux.tar.gz
                    tar -xzf ZAP_2.15.0_Linux.tar.gz
                    ./ZAP_2.15.0/zap.sh -cmd -quickurl https://s410-exam.cyber-ed.space:8084 -quickout $(pwd)/zap-report.json
                '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap-report.json ', allowEmptyArchive: true
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

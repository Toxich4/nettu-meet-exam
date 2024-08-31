pipeline {
    agent any
    
    stages {
        stage('SAST with Semgrep') {
            steps {
                script {
                    sh '''
                    apk add python3 py3-pip py3-virtualenv
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install semgrep
                    semgrep --config=auto . --json > semgrep-report.json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'semgrep-report.json', allowEmptyArchive: true
                }
            }
        }
        stage('SCA with deptrack'){
            steps {
                script{
                    sh '''
                        curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
                        syft dir:$(pwd) -o cyclonedx-json > sbom.json
                    '''
                    sh '''
                        curl -k -X 'PUT' 'https://s410-exam.cyber-ed.space:8081/api/v1/project' \
                             -H 'accept: application/json' \
                             -H 'X-API-Key: odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl' \
                             -H 'Content-Type: application/json' \
                             -d '{
                                   "name": "toxi4",
                                   "version": "1.0.0",
                                   "description": "exam-project"
                                 }'
                        '''
                    sh '''
                        curl -k -X 'POST' 'https://s410-exam.cyber-ed.space:8081/api/v1/bom' \
                        -H 'accept: application/json' \
                        -H 'Content-Type: multipart/form-data'\
                        -H 'X-API-Key: odt_SfCq7Csub3peq7Y6lSlQy5Ngp9sSYpJl' \
                        -F 'projectName=toxi4' \
                        -F 'projectVersion=1.0.0' \
                        -F 'bom=@sbom.json'
                        '''                            
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'sbom.json', allowEmptyArchive: true
                }
            }
        }
        /*stage('SCA with Dependency-Check') {
            agent { label 'dind' }
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
        }*/

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
            agent { label 'dind' }
            steps {
                script {
                    sh '''
                        mkdir ./report
                        docker login -u kafigah430@kwalah.com -p Qwerty123!
                        docker run -v ./report:/report aquasec/trivy repo https://github.com/Bugamed/nettu-meet-exam -f json -o /report/trivy-report.json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'report/trivy-report.json', allowEmptyArchive: true
                }
            } 
        }
            stage('Upload Semgrep Report to DefectDojo') {
            steps {
                script {
                    sh '''
                    curl -k -X 'POST' 'https://s410-exam.cyber-ed.space:8083/api/v2/import-scan/' \
                      -H "accept: application/json" \
                      -H "Content-Type: multipart/form-data" \
                      -H "Authorization: Token c5b50032ffd2e0aa02e2ff56ac23f0e350af75b4"  \
                      -F "file=@semgrep-report.json" \
                      -F "minimum_severity=High" \
                      -F "product_name=Toxi4" \
                      -F "scan_type=Semgrep JSON Report" \
                      -F "engagement=1" \
                      -F "active=true" \
                      -F "verified=true" \
                      -F "tags=semgrep" \
                      -F "group_by=component_name" \
                      -F "test_title=toxi4" \
                      -F "close_old_findings=true" \
                      -F "close_old_findings_product_scope=true" \
                      -F "scan_date=$(date +%F)"
                    '''
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

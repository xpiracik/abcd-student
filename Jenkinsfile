pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github-pat', url: 'https://github.com/xpiracik/abcd-student', branch: 'main'
                }
            }
        }
        stage('Przygotowanie') {
            steps {
                
                sh 'mkdir -p results/'
            }
        }
        stage('Skan Pasywny') {
            steps {
                sh '''
                    docker stop juice-shop || true
                    docker rm juice-shop || true               
                    docker run --name juice-shop -d --rm \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker stop zap || true
                    docker rm zap || true
                    docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /c/DEV/SzkolenieABC/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                        || true 
                '''
                sh '''
                    echo "Checking if report files exist in the container..."
                    docker ps -a
                    docker logs zap || true
                    docker exec zap ls -la /zap/wrk/reports/
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/zap_html_report.html"
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/zap_xml_report.xml"
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                }
            }
        }
    }
}
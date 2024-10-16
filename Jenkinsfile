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
                        -v /c/DEV/SzkolenieABC/abcd-student/.zap:/zap/wrk/:rw \
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
                        || true 
                '''
            }
            post {
                always {
                    sh '''
                        docker cp zap:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop || true
                        docker rm zap || true
                    '''
                }
            }

            post {
            always {
                echo 'Zapisawanie wyniku do  results...'
                archiveArtifacts  artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                // echo 'Wysyłanie raportu do DefectDojo ...'                
                // defectDojoPublisher(artifact: 'results/sca-osv-scanner.json', 
                //     productName: 'Juice Shop', 
                //     scanType: 'OSV Scan', 
                //     engagementName: 'mariusz.klys@symfonia.pl')
            }
        }
        }
    }
}
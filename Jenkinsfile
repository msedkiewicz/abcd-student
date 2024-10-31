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
                    git credentialsId: 'devsecops', url: 'https://github.com/msedkiewicz/abcd-student.git', branch: 'main'
                }
            }
        }
        stage('Prepare report space') {
            steps {
                sh 'mkdir -p results/'
            }
        }
        // stage('[ZAP] Baseline passive-scan') {
        //     steps {
        //         sh '''
        //             docker run --name juice-shop -d --rm \
        //                 -p 3000:3000 bkimminich/juice-shop
        //             sleep 15
        //         '''
        //         sh '''
        //             docker run --name zap \
        //                 --add-host=host.docker.internal:host-gateway \
        //                 -v /mnt/d/devops/abcd-student/.zap:/zap/wrk/:rw  \
        //                 -t ghcr.io/zaproxy/zaproxy:stable bash -c \
        //                 "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" \
        //                 || true
        //         '''
        //     }
        //     post {
        //         always {
        //             sh '''
        //                 docker cp zap:/zap/wrk/reports/zap_html_report.html "${WORKSPACE}/results/zap_html_report.html"
        //                 docker cp zap:/zap/wrk/reports/zap_xml_report.xml "${WORKSPACE}/results/zap_xml_report.xml"
        //                 docker stop zap
        //                 docker rm zap
        //             '''
        //             echo 'Archiving results'
        //             archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
        //             echo 'Sending ZAP passive scan reports to DefectDojo'
        //             defectDojoPublisher(artifact: 'results/zap_xml_report.xml', productName: 'Juice Shop', scanType: 'ZAP Scan', engagementName: 'magdalenainspirations@gmail.com')
        //         }
        //     }
        // }
        stage('OSV-Scanner') {
            steps {
                sh '''
                osv-scanner scan --lockfile package-lock.json --json --output "${WORKSPACE}/results/osv-report.json"  || true
                '''
            }
            post {
                always {
                    echo 'Archiving results'
                    archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
                    echo 'Sending OSV scan report to DefectDojo'
                    defectDojoPublisher(artifact: 'results/osv-report.json', productName: 'Juice Shop', scanType: 'OSV Scan', engagementName: 'magdalenainspirations@gmail.com')
                }
            }
        }

        stage('Trufflehog') {
            steps {
                sh '''
                    trufflehog git file://. --branch master --only-verified --json "${WORKSPACE}/results/trufflehog-results.json"
                '''
            }
            // post {
            //     always {
            //         echo 'Archiving results'
            //         archiveArtifacts artifacts: 'results/**/*', fingerprint: true, allowEmptyArchive: true
            //         echo 'Sending Trufflehog report to DefectDojo'
            //         defectDojoPublisher(artifact: 'results/trufflehog-results.json', productName: 'Juice Shop', scanType: 'Trufflehog Scan', engagementName: 'magdalenainspirations@gmail.com')
            //     }
            // }
        }
        stage('Cleaning') {
            steps {
                sh '''
                    docker stop juice-shop
                '''
            }
            post {
                always {
                    echo 'cleaning complete'
                }
            }
        }
    }
}
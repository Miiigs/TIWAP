pipeline {
    agent any

    environment {
        JENKINS_URL = 'http://localhost:8080' // URL de Jenkins
        ZAP_API = 'https://172.17.0.2:8000'    // URL de la API de OWASP ZAP
        ZAP_API_KEY = '123456789'             // API Key de OWASP ZAP
        TARGET_URL = 'https://172.17.0.1:5000' // URL de la aplicación TIWAP
    }

    stages {
        stage('Run Spider Scan') {
            steps {
                echo 'Ejecutando Spider Scan con OWASP ZAP...'
                sh '''
                curl -k -X POST \
                    -d "apikey=${ZAP_API_KEY}" \
                    -d "url=${TARGET_URL}" \
                    ${ZAP_API}/JSON/spider/action/scan/
                echo "Esperando a que el Spider Scan finalice..."
                while true; do
                    response=$(curl -k -s ${ZAP_API}/JSON/spider/view/status/?apikey=${ZAP_API_KEY})
                    status=$(echo "$response" | grep -o '"status":"[0-9]*"' | sed 's/[^0-9]//g')
                    if [ "$status" -eq 100 ]; then
                        break
                    fi
                    echo "Progreso del Spider Scan: $status%"
                    sleep 5
                done
                '''
            }
        }

        stage('Run Active Scan') {
            steps {
                echo 'Ejecutando Active Scan con OWASP ZAP...'
                sh '''
                curl -k -X POST \
                    -d "apikey=${ZAP_API_KEY}" \
                    -d "url=${TARGET_URL}" \
                    ${ZAP_API}/JSON/ascan/action/scan/
                echo "Esperando a que el Active Scan finalice..."
                while true; do
                    response=$(curl -k -s ${ZAP_API}/JSON/ascan/view/status/?apikey=${ZAP_API_KEY})
                    status=$(echo "$response" | grep -o '"status":"[0-9]*"' | sed 's/[^0-9]//g')
                    if [ "$status" -eq 100 ]; then
                        break
                    fi
                    echo "Progreso del Active Scan: $status%"
                    sleep 5
                done
                '''
            }
        }

        stage('Collect Reports') {
            steps {
                echo 'Recopilando reportes generados por OWASP ZAP...'
                sh '''
                curl -k -s ${ZAP_API}/OTHER/core/other/htmlreport/?apikey=${ZAP_API_KEY} > zap_report.html
                '''
                archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
            }
        }
    }
}

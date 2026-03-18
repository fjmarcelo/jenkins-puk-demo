pipeline {
    agent any

    environment {
        IMAGE_NAME = "secureapi"
        IMAGE_TAG  = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Código obtenido de GitHub"
                sh 'ls -la'
            }
        }

        stage('SAST - Bandit') {
            steps {
                echo "Análisis estático de seguridad con Bandit"
                sh '''
                    docker run --rm \
                        -v ${WORKSPACE}:/src \
                        cytopia/bandit \
                        -r /src/app \
                        -f txt \
                        -o /src/bandit-report.txt \
                        || true
                    cat bandit-report.txt
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-report.txt', allowEmptyArchive: true
                }
            }
        }

        stage('SCA - pip-audit') {
            steps {
                echo "Análisis de dependencias con pip-audit"
                sh '''
                    docker run --rm \
                        -v ${WORKSPACE}:/src \
                        pypa/pip-audit \
                        -r /src/requirements.txt \
                        -f json \
                        -o /src/pip-audit-report.json \
                        || true
                    cat pip-audit-report.json
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'pip-audit-report.json', allowEmptyArchive: true
                }
            }
        }

        stage('Build imagen Docker') {
            steps {
                echo "Construyendo imagen ${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Escaneo imagen - Trivy') {
            steps {
                echo "Escaneando imagen con Trivy"
                sh '''
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v ${WORKSPACE}:/output \
                        aquasec/trivy image \
                        --format table \
                        --output /output/trivy-report.txt \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        || true
                    cat trivy-report.txt
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline completado. Revisa los artefactos para los informes de seguridad."
        }
        failure {
            echo "Pipeline fallido. Revisa el Console Output."
        }
    }
}

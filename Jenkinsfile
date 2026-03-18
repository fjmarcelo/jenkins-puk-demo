pipeline {
    agent any

    environment {
        IMAGE_NAME = "secureapi"
        IMAGE_TAG  = "build-${BUILD_NUMBER}"
    }

    stages {

        // stage('Checkout') {
        //     steps {
        //         cleanWs()
        //         checkout scm
        //         echo "Código obtenido de GitHub"
        //         sh 'ls -la && ls -la app/'
        //     }
        // }
        //
        // stage('SAST - Bandit') {
        //     steps {
        //         echo "Análisis estático de seguridad con Bandit"
        //         sh '''
        //             docker run --rm \
        //                 -v ${WORKSPACE}:/src \
        //                 python:3.12-slim \
        //                 sh -c "pip install bandit -q && bandit -r /src/app -f txt || true"
        //         '''
        //     }
        // }
        // stage('SAST - Bandit') {
        //     steps {
        //         echo "Análisis estático de seguridad con Bandit"
        //         sh """
        //             docker run --rm \\
        //                 -v \${WORKSPACE}:/src \\
        //                 python:3.12-slim \\
        //                 sh -c 'pip install bandit -q && bandit -r /src/app -f txt || true'
        //         """
        //     }
        // }

        stage('SCA - pip-audit') {
            steps {
                echo "Análisis de dependencias con pip-audit"
                sh '''
                    docker run --rm \
                        -v ${WORKSPACE}:/src \
                        ghcr.io/pypa/pip-audit \
                        -r /src/requirements.txt \
                        || true
                '''
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
                        aquasec/trivy image \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${IMAGE_TAG} \
                        || true
                '''
            }
        }

    }

    post {
        success {
            echo "Pipeline completado. Revisa el Console Output para los informes de seguridad."
        }
        failure {
            echo "Pipeline fallido. Revisa el Console Output."
        }
    }
}

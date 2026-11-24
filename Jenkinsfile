pipeline {
    agent { label 'ci-python-docker' }
    options {
        timeout(time: 15, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Tests') {
            steps {
                // TODO: criar .venv, instalar requirements*.txt e executar pytest -q
                sh '''
                    python3 -m venv .venv
                    .venv/bin/python -m pip install -r requirements.txt -r requirements-dev.txt
                    .venv/bin/python -m pytest -q
                '''
                echo 'TODO: executar testes Python'
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                // TODO: docker build com tag natixis-support:ci-${BUILD_NUMBER}
                docker build -t "natixis-support:ci-${BUILD_NUMBER}" .
                // TODO: confirmar imagem criada com docker image inspect
                docker image inspect "natixis-support:ci-${BUILD_NUMBER}" --format '{{.Id}}'
                '''
                echo 'TODO: construir e verificar imagem Docker'
            }
        }
    }
}

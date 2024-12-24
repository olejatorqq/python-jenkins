pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "oorlovsk/flask-jenkins"
        DOCKER_TAG = "build-${env.BUILD_NUMBER}"
    }

    stages{
        stage('Checkout') {
            steps{
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh """
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                """
            }
        }

        stage('Test') {
            steps {
                sh 'pytest --maxfail=1 --disable-warnings'
            }
        }

        stage('Build & Push Docker') {
            when {
                expression { return env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    // Логинимся в Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds',
                                                      usernameVariable: 'DOCKER_USER',
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    }
                    // Собираем образ
                    sh "docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ."
                    // Пушим образ с конкретным тегом
                    sh "docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                      docker ps -q --filter "name=jenkins-flask" | xargs -r docker stop
                      docker run -d --rm --name jenkins-flask -p 5000:5000 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    """
                }
            }
        }
    }
}
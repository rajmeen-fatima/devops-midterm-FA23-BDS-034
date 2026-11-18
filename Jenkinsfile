
pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_NAME = 'midterm-fa23-bds-034'
        CONTAINER_NAME = 'midterm-api'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv
                    .venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Train Model') {
            steps {
                sh '''
                    GIT_COMMIT=$(git rev-parse HEAD) \
                    .venv/bin/python train.py
                '''
            }
        }

        stage('Test Model') {
            steps {
                sh '''
                    .venv/bin/python -m compileall -q app.py train.py
                    .venv/bin/python -c "import json; m=json.load(open('artifacts/metrics.json')); assert m['training_rows'] >= 100; assert 0 <= m['accuracy'] <= 1"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true
                    docker run -d \
                      --restart unless-stopped \
                      --name $CONTAINER_NAME \
                      -p 8000:8000 \
                      $IMAGE_NAME:latest
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    for i in 1 2 3 4 5 6 7 8 9 10
                    do
                        curl -fsS http://localhost:8000/metrics && exit 0
                        sleep 2
                    done
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            mail(
                to: 'qasimalik@gmail.com',
                subject: "DevOps Midterm FA23-BDS-034 - ${currentBuild.currentResult}",
                body: """Student: FA23-BDS-034
Build: ${env.BUILD_NUMBER}
Status: ${currentBuild.currentResult}
Commit: ${env.GIT_COMMIT}
Jenkins Job: ${env.BUILD_URL}
"""
            )
        }
    }
}

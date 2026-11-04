pipeline {
    agent any

    environment {
        IMAGE_NAME = 'tacker115/ci-demo'
    }

    stages {

        stage('Lint') {
            steps {
                sh '''
                    python3 -m venv .jenkins-venv
                    .jenkins-venv/bin/pip install -r requirements.txt
                    .jenkins-venv/bin/flake8 app.py test_app.py --select=F
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                sh '.jenkins-venv/bin/pytest -v'
            }
        }

        stage('Security Scan') {
            steps {
                sh '.jenkins-venv/bin/bandit -q -r app.py -s B104'
            }
        }

        stage('DB Migration') {
            steps {
                sh '.jenkins-venv/bin/alembic upgrade head'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ci-demo-deploy || true
                    docker run -d --name ci-demo-deploy -p 5001:5000 $IMAGE_NAME:latest
                    sleep 2
                    curl http://localhost:5001
                '''
            }
        }
    }

    post {
        success {
            sh 'echo "Pipeline SUCCESS" | mail -s "Jenkins ci-demo SUCCESS" tacker'
        }
        failure {
            sh 'echo "Pipeline FAILURE" | mail -s "Jenkins ci-demo FAILURE" tacker'
        }
    }
}

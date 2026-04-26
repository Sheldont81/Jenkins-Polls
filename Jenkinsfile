pipeline {
    agent any
    environment {
        PROJECT_DIR = "/home/ubuntu/Jenkins-Polls"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "sheldont81/django-polls"
    }
    triggers {
        githubPush()
    }
    stages {
        stage('Update Code') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    git pull origin main
                """
            }
        }
        stage('Build Docker Image') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh """
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${IMAGE_NAME}:latest
                """
            }
        }
        stage('Deploy') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker stop django_app nginx_proxy 2>/dev/null || true
                    docker rm django_app nginx_proxy 2>/dev/null || true
                    docker compose up -d
                """
            }
        }
        stage('Run Migrations') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker compose exec -T web python manage.py migrate --noinput
                """
            }
        }
        stage('Collect Static Files') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker compose exec -T web python manage.py collectstatic --noinput
                """
            }
        }
        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
    post {
        success {
            echo "Successfully deployed and pushed to Docker Hub!"
        }
        failure {
            echo "Deployment failed. Check Jenkins console output."
        }
    }
}
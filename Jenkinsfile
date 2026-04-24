pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/django_polls"
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

        stage('Build and Deploy') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker-compose up -d --build
                """
            }
        }

        stage('Run Migrations') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker-compose exec -T web python manage.py migrate --noinput
                """
            }
        }

        stage('Collect Static Files') {
            steps {
                sh """
                    cd ${PROJECT_DIR}
                    docker-compose exec -T web python manage.py collectstatic --noinput
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
            echo "Successfully deployed Dockerized Django app to AWS!"
        }
        failure {
            echo "Deployment failed. Check Jenkins console output."
        }
    }
}
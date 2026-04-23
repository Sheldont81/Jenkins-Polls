pipeline {
    agent any

    environment {
        EC2_USER    = "ubuntu"
        EC2_HOST    = "3.141.190.68"
        EC2_KEY     = credentials('ec2-ssh-private-key')
        PROJECT_DIR = "/home/ubuntu/django_polls"
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Dockerized Deployment') {
            steps {
                script {
                    sshagent (credentials: ['ec2-ssh-private-key']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
                            set -e
                            cd ${PROJECT_DIR}
                            echo "Updating code..."
                            git pull origin main
                            echo "Rebuilding containers..."
                            sudo docker-compose up -d --build
                            echo "Running database migrations..."
                            sudo docker-compose exec -T web python manage.py migrate --noinput
                            echo "Collecting static files..."
                            sudo docker-compose exec -T web python manage.py collectstatic --noinput
                            sudo docker image prune -f
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed Dockerized Django app to AWS!"
        }
        failure {
            echo "Deployment failed. Check Jenkins console and AWS Security Groups."
        }
    }
}
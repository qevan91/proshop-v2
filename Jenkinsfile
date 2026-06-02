pipeline {
    agent any
    environment {
        PROJECT_DIR = "/workspace/proshop-v2"
        DOCKER_USER = "Evan"
    }
    stages {
        stage('Informations') {
            steps {
                echo '===== Informations ====='
                sh 'pwd && ls -la $PROJECT_DIR'
            }
        }
        stage('Docker Test') {
            steps {
                echo '===== Verification Docker ====='
                sh 'docker --version && docker ps'
            }
        }
        stage('Build & Tag Images') {
            steps {
                echo '===== Build & Tag du Backend et Frontend ====='
                sh '''
                cd $PROJECT_DIR
                # Build et Tag v1.0 + latest pour le Backend
                docker build -t proshop-backend:latest -t proshop-backend:v1.0 -f backend/Dockerfile .
                docker tag proshop-backend:v1.0 $DOCKER_USER/proshop-backend:v1.0
                docker tag proshop-backend:latest $DOCKER_USER/proshop-backend:latest

                # Build et Tag v1.0 + latest pour le Frontend
                docker build -t proshop-frontend:latest -t proshop-frontend:v1.0 frontend/
                docker tag proshop-frontend:v1.0 $DOCKER_USER/proshop-frontend:v1.0
                docker tag proshop-frontend:latest $DOCKER_USER/proshop-frontend:latest
                '''
            }
        }
        stage('Push to Registry') {
            steps {
                echo '===== Connexion et Push vers Docker Hub ====='
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin

                    # Push Backend
                    docker push $DOCKER_USER/proshop-backend:v1.0
                    docker push $DOCKER_USER/proshop-backend:latest

                    # Push Frontend
                    docker push $DOCKER_USER/proshop-frontend:v1.0
                    docker push $DOCKER_USER/proshop-frontend:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                echo '===== Deploiement ====='
                sh '''
                cd $PROJECT_DIR
                docker-compose down || true
                docker-compose up -d --build
                '''
            }
        }
        stage('Verify') {
            steps {
                echo '===== Verification ====='
                sh '''
                cd $PROJECT_DIR
                docker ps
                echo "--- Statut des services Compose ---"
                docker-compose ps
                '''
            }
        }
    }
    post {
        success {
            echo '================================='
            echo 'PIPELINE EXECUTE AVEC SUCCES'
            echo '================================='
            echo 'Frontend : http://localhost:3000'
            echo 'Backend  : http://localhost:5000'
            echo 'MongoDB  : localhost:27017'
        }
        failure {
            echo '================================='
            echo 'ECHEC DU PIPELINE'
            echo '================================='
        }
    }
}
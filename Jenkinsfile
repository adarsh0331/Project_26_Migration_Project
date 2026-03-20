pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "adarshbarkunta/python-flask-app"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/adarsh0331/Project_26_Migration_Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push Docker Image') {
            steps {
              withCredentials([usernamePassword(
              credentialsId: 'dockerhub',
              usernameVariable: 'USERNAME',
              passwordVariable: 'PASSWORD')]) {

            sh '''
            echo $PASSWORD | docker login -u $USERNAME --password-stdin
            docker push adarshbarkunta/python-flask-app:latest
            '''
        }
    }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop python-app || true
                docker rm python-app || true
                docker run -d -p 5000:5000 --name python-app $DOCKER_IMAGE:latest
                '''
            }
        }
    }
}

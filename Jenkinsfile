pipeline {
    agent any

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/ShrinivasC96/Devops-Hotstar-Clone-Project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t hotstar-app .'
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh 'docker rm -f hotstar-container || true'
                    sh 'docker run -d -p 80:80 --name hotstar-container hotstar-app'
                }
            }
        }
    }
}

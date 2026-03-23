pipeline {
    agent any

    tools {
        sonarRunner 'sonar-scanner'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/ShrinivasC96/Devops-Hotstar-Clone-Project.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=hotstar \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://13.235.8.236:9000 \
                    -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t hotstar-app .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f hotstar-container || true'
                sh 'docker run -d -p 80:3000 --name hotstar-container hotstar-app'
            }
        }
    }
}

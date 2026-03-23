pipeline {
    agent any

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/ShrinivasC96/Devops-Hotstar-Clone-Project.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
            }
        }

        stage('Test') {
            steps {
                echo 'Testing project...'
            }
        }
    }
}

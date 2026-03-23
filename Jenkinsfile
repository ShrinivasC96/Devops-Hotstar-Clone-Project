pipeline {
    agent any

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/<your-username>/hotstar-devsecops.git'
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

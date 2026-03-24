pipeline {
    agent any
 
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
                    docker run --rm \
                    --network devsecops-net \
                    -v $(pwd):/usr/src \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=hotstar \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://sonar:9000 \
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

        stage('Trivy Scan') {
            steps {
                sh 'trivy image hotstar-app'
            }
        }
    
        // stage('OWASP Scan') {
        //     steps {
        //         sh '''
        //         docker run --rm \
        //         -v $(pwd):/src \
        //         owasp/dependency-check \
        //         --scan /src \
        //         --format XML \
        //         --out /src
        //         '''
        //     }
        // }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker tag hotstar-app $DOCKER_USER/hotstar-app:latest
                    docker push $DOCKER_USER/hotstar-app:latest
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f hotstar-container || true'
                sh 'docker run -d -p 80:3000 --name hotstar-container hotstar-app'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/.kube/config
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }
}

pipeline {
    agent any

    environment {
        IMAGE_NAME = "hotstar-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_HUB = credentials('docker-creds')
        SONAR_TOKEN = credentials('sonar-token')
    }

    options {
        disableConcurrentBuilds()
        timestamps()
        timeout(time: 45, unit: 'MINUTES')
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/ShrinivasC96/Devops-Hotstar-Clone-Project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('SonarQube Scan (SAST)') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    docker run --rm \
                    -v $WORKSPACE:/usr/src \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=hotstar \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=$SONAR_HOST_URL \
                    -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Scan (Image Scan)') {
            steps {
                sh '''
                trivy image --severity HIGH,CRITICAL --exit-code 1 $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Tag & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    docker tag $IMAGE_NAME:$IMAGE_TAG $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                sh '''
                export KUBECONFIG=/var/lib/jenkins/.kube/config

                # Update image dynamically
                sed -i "s|image: .*|image: $DOCKER_USER/$IMAGE_NAME:$IMAGE_TAG|g" deployment.yaml

                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "Checking Pods..."
                kubectl get pods -o wide

                echo "Checking Services..."
                kubectl get svc

                echo "Checking Deployment Status..."
                kubectl rollout status deployment/hotstar-deployment
                '''
            }
        }

        stage('Fetch App URL') {
            steps {
                script {
                    env.APP_URL = sh(
                        script: "kubectl get svc hotstar-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
                script {
                    if (!env.APP_URL) {
                        error "App URL not found!"
                    }

                    sh '''
                    docker run --rm \
                    ghcr.io/zaproxy/zaproxy:stable \
                    zap-baseline.py \
                    -t http://$APP_URL \
                    -m 5
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
            echo "🌐 Application URL: http://${APP_URL}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
        always {
            echo "🔁 Pipeline finished."
        }
    }
}

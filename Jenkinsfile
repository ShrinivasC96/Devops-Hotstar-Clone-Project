pipeline {
    agent any

    environment {
        IMAGE_NAME = "hotstar-app"
        IMAGE_TAG = "1.0"
        DOCKER_CREDS = credentials('docker-creds')
        DOCKER_USER = "${DOCKER_CREDS_USR}"
        DOCKER_PASS = "${DOCKER_CREDS_PSW}"
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
                trivy image --severity CRITICAL --exit-code 1 $IMAGE_NAME:$IMAGE_TAG || true
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

        stage('Deploy to EKS') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'AWS_Access_Key',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region ap-south-1

                        kubectl apply -f namespace.yaml

                        sed -i "s|image: .*|image: $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG|g" deployment.yaml

                        echo "=== deployment.yaml ==="
                        cat deployment.yaml

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                    '''
                }
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
                    docker run --rm -u 0 -v $(pwd):/zap/wrk/:rw ghcr.io/zaproxy/zaproxy:stable \
                        zap-baseline.py -t http://$APP_URL -m 5 -I -r zap_report.html
                    '''
                }
            }
        }
        
        stage('Publish ZAP Report') {
            steps {
                publishHTML(target: [
                    reportDir: '.',   // current directory where report is generated
                    reportFiles: 'zap_report.html',
                    reportName: 'OWASP ZAP Report'
                ])
                archiveArtifacts artifacts: "zap_report.html"
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

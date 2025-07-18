pipeline {
    agent any

    environment {
        APP_NAME = 'my-app'
        BUILD_DIR = 'build'
        DOCKER_IMAGE = "myrepo/${APP_NAME}:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                git url: 'https://github.com/your-org/your-repo.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                echo "Building the application..."
                sh './gradlew build'
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                sh './gradlew test'
            }
        }

        stage('Code Quality') {
            steps {
                echo "Running SonarQube analysis..."
                withSonarQubeEnv('SonarQubeServer') {
                    sh './gradlew sonarqube'
                }
            }
        }

        stage('Package') {
            steps {
                echo "Packaging application..."
                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo "Pushing Docker image to registry..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                echo "Deploying to Dev environment..."
                // sh 'kubectl apply -f k8s/dev-deployment.yaml'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Sending notifications...'
            // emailext or slackSend
        }
        always {
            cleanWs()
        }
    }
}

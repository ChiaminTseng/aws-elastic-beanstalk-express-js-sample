pipeline {
    // Pipeline configuration
    agent none // Use specific agents for each stage
    environment {
        // Set the Docker image name and tag for the app
        IMAGE_NAME = 'chiamintwts/assignment2_22165266:latest'
    }
    stages {
        // Stage 1: Install Node.js dependencies
        stage('Install Dependencies') {
            agent { docker { image 'node:16' } } // Use Node 16 Docker image for build agent
            steps {
                // Install dependencies using npm
                sh 'npm install'
            }
        }
        // Stage 2: Run unit tests
        stage('Run Unit Tests') {
            agent { docker { image 'node:16' } }
            steps {
                // Run the test suite
                sh 'npm test'
            }
        }
        // Stage 3: Security scan with Snyk
        stage('Security Scan') {
            agent { docker { image 'node:16' } }
            steps {
                // Install Snyk CLI and run a scan
                sh 'npm install -g snyk'
                // Pipeline fails if any high severity issues are found
                sh 'snyk test --severity-threshold=high'
            }
        }
        // Stage 4: Build Docker image for the app
        stage('Build Docker Image') {
            agent any // Run on Jenkins master
            steps {
                // Build the Docker image using the Dockerfile
                sh 'docker build -t $IMAGE_NAME .'
            }
        }
        // Stage 5: Push Docker image to Docker Hub
        stage('Push Docker Image') {
            agent any
            steps {
                // Login to Docker Hub and push the image
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }
        // Stage 6: Create build log file
        stage('Create Log') {
            agent any
            steps {
                // Create a log file with pipeline execution timestamp
                sh 'echo "Pipeline ran successfully on $(date)" > build.log'
            }
        }
    }
    // Archive build log as artifact
        post {
            always {
                // Ensure workspace context for archiving
                archiveArtifacts artifacts: 'build.log', allowEmptyArchive: true
            }
        }
}
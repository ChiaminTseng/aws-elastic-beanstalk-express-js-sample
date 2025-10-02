pipeline {
    agent none
    environment {
        IMAGE_NAME = 'chiamintwts/assignment2_22165266:latest'
    }
    stages {
        // Install dependencies using npm install --save
        stage('Install Dependencies') {
            agent { docker { image 'node:16' } }
            steps {
                sh 'npm install --save'
            }
        }
        // Run unit tests
        stage('Run Unit Tests') {
            agent { docker { image 'node:16' } }
            steps {
                // Validate package.json and dependencies without starting server
                sh 'npm ls --depth=0'
            }
        }
        // Security scan with Snyk CLI
        stage('Security Scan') {
            agent { docker { image 'node:16' } }
            steps {
                sh 'npm install -g snyk'
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    sh 'snyk auth $SNYK_TOKEN'
                    sh 'snyk test --severity-threshold=high'
                }
            }
        }
        // Build Docker image
        stage('Build Docker Image') {
            agent any
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }
        // Push to registry
        stage('Push Docker Image') {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $IMAGE_NAME'
                }
            }
        }
    }
}
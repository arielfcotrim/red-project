pipeline {
    // Define environment variables
    environment {
        // Image names for the backend and frontend
        BACKEND_IMAGE = "red_proj_backend:v1"
        FRONTEND_IMAGE = "red_proj_frontend:v1"
        // Docker Hub login credentials
        DOCKER_USERNAME = credentials('docker_username')
        DOCKER_PASSWORD = credentials('docker_password')
        // AWS credentials for Terraform
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }

    // run on any available Jenkins agent
    agent any
    stages {
        stage('Build') {
            steps {
                // Build steps for the backend
                dir('server') {
                    sh 'npm install'
                }

                // Build steps for the frontend
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                // Go into the tests directory
                dir('test') {
                    // Install requirements
                    sh 'pip install -r requirements.txt'
                    // Run the test.py file
                    sh 'python3 -m pytest test.py'
                }
            }
        }

        stage('Deployment') {
            steps {
                // Build the Docker images with the Docker Hub username and repository included in the image name
                sh "docker build -t $DOCKER_USERNAME/$BACKEND_IMAGE server"
                sh "docker build -t $DOCKER_USERNAME/$FRONTEND_IMAGE frontend"
                // Log in to Docker Hub
                sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                // Push the images to Docker Hub
                sh "docker push $DOCKER_USERNAME/$BACKEND_IMAGE"
                sh "docker push $DOCKER_USERNAME/$FRONTEND_IMAGE"
            }
        }

        stage('Delivery') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                    withEnv([
                        ${env.AWS_ACCESS_KEY_ID},
                        ${env.AWS_SECRET_ACCESS_KEY},
                        ${env.DOCKER_USERNAME},
                        ${env.BACKEND_IMAGE},
                        ${env.FRONTEND_IMAGE}
                    ])
                }
            }
        }
    }
}

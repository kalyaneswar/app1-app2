pipeline {
    // Use the Jenkins agent labeled 'AGENT-1'
    agent { label 'AGENT-1' }

    environment {
        // Docker image names
        APP1_IMAGE = "app1-image"
        APP2_IMAGE = "app2-image"

        // Image version tags
        VERSION_APP1 = "v1.0"
        VERSION_APP2 = "v1.0"

        // Docker Hub repository name
        DOCKER_REPO = "kalyaneswarm"
    }

    stages {

        stage('Docker Login') {
            steps {
                // Login to Docker Hub using secure credentials from Jenkins credentials store
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                // Checkout the source code from the Git repository configured in the job
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Build the Docker image for app1 using the Dockerfile inside the 'app1' directory
                    dir('app1') {
                        sh 'docker build -t $APP1_IMAGE:$VERSION_APP1 .'
                    }

                    // Build the Docker image for app2 using the Dockerfile inside the 'app2' directory
                    dir('app2') {
                        sh 'docker build -t $APP2_IMAGE:$VERSION_APP2 .'
                    }
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                script {
                    // Tag images with Docker Hub repo name for pushing
                    sh '''
                        docker tag $APP1_IMAGE:$VERSION_APP1 $DOCKER_REPO/$APP1_IMAGE:$VERSION_APP1
                        docker tag $APP2_IMAGE:$VERSION_APP2 $DOCKER_REPO/$APP2_IMAGE:$VERSION_APP2
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    // Push the tagged images to Docker Hub repository
                    sh 'docker push $DOCKER_REPO/$APP1_IMAGE:$VERSION_APP1'
                    sh 'docker push $DOCKER_REPO/$APP2_IMAGE:$VERSION_APP2'
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    // Stop and remove any existing containers (ignore errors)
                    sh '''
                        docker rm -f app1-container || true
                        docker rm -f app2-container || true
                    '''

                    // Run app1 container on port 8081
                    sh 'docker run -d -p 8081:80 --name app1-container $DOCKER_REPO/$APP1_IMAGE:$VERSION_APP1'

                    // Run app2 container on port 8082
                    sh 'docker run -d -p 8082:80 --name app2-container $DOCKER_REPO/$APP2_IMAGE:$VERSION_APP2'
                }
            }
        }
    }

    post {
        // If the pipeline succeeds
        success {
            echo '✅ Docker containers are running on AGENT-1!'
        }

        // If the pipeline fails
        failure {
            echo '❌ Something went wrong during the pipeline.'
        }
    }
}

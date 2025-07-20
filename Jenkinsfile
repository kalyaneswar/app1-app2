pipeline {
    // Run the pipeline only on the agent labeled 'AGENT-1'
    agent { label 'AGENT-1' }

    environment {
        // Define Docker image names as environment variables
        APP1_IMAGE = "app1-image"
        APP2_IMAGE = "app2-image"
        VERSION_APP1 = "v1.0"
        VERSION_APP2 = "v1.0"
    }

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) 
                {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }
    }

        stages {
            stage('Checkout Code') {
                steps {
                    // Pull the latest code from the Git repository
                    checkout scm
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Navigate into the app1 directory and build Docker image
                    dir('app1') {
                        sh 'docker build -t $APP1_IMAGE:$VERSION_APP1 .'
                    }

                    // Navigate into the app2 directory and build Docker image
                    dir('app2') {
                        sh 'docker build -t $APP2_IMAGE:$VERSION_APP2 .'
                    }
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    // Stop and remove containers if already running (ignore errors)
                    sh '''
                        docker rm -f app1-container || true
                        docker rm -f app2-container || true
                    '''

                    // Run app1 container on port 8081
                    sh 'docker run -d -p 8081:80 --name app1-container $APP1_IMAGE:$VERSION_APP1'

                    // Run app2 container on port 8082
                    sh 'docker run -d -p 8082:80 --name app2-container $APP2_IMAGE:$VERSION_APP2'
                }
            }
        }

        stage('Tag images with Docker repository ') {
            steps {
                script{
                    sh ''' 
                        docker tag $APP1_IMAGE:$VERSION_APP1 kalyaneswarm/$APP1_IMAGE:$VERSION_APP1
                        docker tag $APP2_IMAGE:$VERSION_APP2 kalyaneswarm/$APP2_IMAGE:$VERSION_APP2
                    '''
                }
            }
        }

        stage("Push docker images to docker repository") {
            steps{
                script{
                    sh 'docker push kalyaneswarm/$APP1_IMAGE:$VERSION_APP1'  
                    sh 'docker push kalyaneswarm/$APP2_IMAGE:$VERSION_APP2'
                                                               
                }
            }
        }
    }

    post {
        // Message if pipeline succeeds
        success {
            echo '✅ Docker containers are running on AGENT-1!'
        }

        // Message if pipeline fails
        failure {
            echo '❌ Something went wrong during the pipeline.'
        }
    }
}

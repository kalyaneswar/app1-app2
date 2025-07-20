pipeline {
    agent { label 'AGENT-1' }

    environment {
        APP1_IMAGE = "app1-image"
        APP2_IMAGE = "app2-image"
        VERSION_APP1 = "v1.0"
        VERSION_APP2 = "v1.0"
    }

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    dir('app1') {
                        sh 'docker build -t $APP1_IMAGE:$VERSION_APP1 .'
                    }
                    dir('app2') {
                        sh 'docker build -t $APP2_IMAGE:$VERSION_APP2 .'
                    }
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    sh '''
                        docker rm -f app1-container || true
                        docker rm -f app2-container || true
                    '''
                    sh 'docker run -d -p 8081:80 --name app1-container $APP1_IMAGE:$VERSION_APP1'
                    sh 'docker run -d -p 8082:80 --name app2-container $APP2_IMAGE:$VERSION_APP2'
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                script {
                    sh '''
                        docker tag $APP1_IMAGE:$VERSION_APP1 kalyaneswarm/$APP1_IMAGE:$VERSION_APP1
                        docker tag $APP2_IMAGE:$VERSION_APP2 kalyaneswarm/$APP2_IMAGE:$VERSION_APP2
                    '''
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh 'docker push kalyaneswarm/$APP1_IMAGE:$VERSION_APP1'
                    sh 'docker push kalyaneswarm/$APP2_IMAGE:$VERSION_APP2'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Docker containers are running on AGENT-1!'
        }
        failure {
            echo '❌ Something went wrong during the pipeline.'
        }
    }
}

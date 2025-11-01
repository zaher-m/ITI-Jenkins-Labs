@Library('jenkins-shared-library@main') _
pipeline {
    agent {
        node { label 'VM_Ubuntu' }
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        IMAGE_NAME = 'm1zaher/jenkins_day02'
    }

    tools {
        maven 'maven-3.5.2'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo "Build Number: ${currentBuild.number}"
                    if (currentBuild.number < 5) {
                        error("Build number < 5. exiting...")
                    }
                    buildM()
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    buildD(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        login(DOCKER_USERNAME, DOCKER_PASSWORD)
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    echo "Pushing Docker image ${IMAGE_NAME}:${BUILD_NUMBER}..."
                    push(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    test()
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    deploy(IMAGE_NAME, BUILD_NUMBER, 'jenkins_lab02', '9000:8080')
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }

        success {
            echo 'Build, test, and deployment were successful!'
        }

        failure {
            echo 'There was an error'
        }

        cleanup {
            echo "Cleaning up Docker containers..."
            sh "docker rm -f jenkins_lab02 || true"
            sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
        }
    }
}
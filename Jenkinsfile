@Library('jenkins-shared-library@main') _

pipeline {
    agent {
        node { label 'VM_Ubuntu' }
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        IMAGE_NAME = 'm1zaher/jenkins_day02'
        BUILD_NUMBER = "${currentBuild.number}"
    }

    tools {
        maven 'maven-3.5.2'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo "Build Number: ${BUILD_NUMBER}"
                    if (currentBuild.number < 5) {
                        error("Build number < 5. Exiting...")
                    }

                    def maven = new edu.iti.Maven()
                    maven.buildM()
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def docker = new edu.iti.Docker()
                    docker.buildD(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        def docker = new edu.iti.Docker()
                        docker.login(DOCKER_USERNAME, DOCKER_PASSWORD)
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    echo "Pushing Docker image ${IMAGE_NAME}:${BUILD_NUMBER}..."

                    def docker = new edu.iti.Docker()
                    docker.push(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def maven = new edu.iti.Maven()
                    maven.test()
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def docker = new edu.iti.Docker()
                    docker.deploy(IMAGE_NAME, BUILD_NUMBER, 'jenkins_lab02', '9000:8080')
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

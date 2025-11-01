@Library('jenkins-shared-library') _ 
    agent {
        node { label "VM_Ubuntu" }
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        IMAGE_NAME = "m1zaher/jenkins_day02"
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
                    // Use the Maven build function from Maven.groovy
                    edu.iti.Maven.mavenBuild()
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Use the Docker build function from Docker.groovy
                    edu.iti.Docker.build(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    // Use the Docker login function from Docker.groovy
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        edu.iti.Docker.login(DOCKER_USERNAME, DOCKER_PASSWORD)
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    echo "Pushing Docker image ${IMAGE_NAME}:${BUILD_NUMBER}..."
                    // Use the Docker push function from Docker.groovy
                    edu.iti.Docker.push(IMAGE_NAME, BUILD_NUMBER)
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Use the Maven test function from Maven.groovy
                    edu.iti.Maven.mavenTest()
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Use the Docker deploy function from Docker.groovy
                    edu.iti.Docker.deploy(IMAGE_NAME, BUILD_NUMBER, 'jenkins_lab02', '9000:8080')
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

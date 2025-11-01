pipeline {
    agent {
        node { label "VM_Ubuntu" }
    }
    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
    }

    stages {
   

        stage('Build') {
            steps {
                script {
                    sh 'mvn package install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh 'echo deploying'
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
    }
}

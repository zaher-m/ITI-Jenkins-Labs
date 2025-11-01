node('VM_Ubuntu') {
    env.JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
    env.IMAGE_NAME = 'm1zaher/jenkins_day02'

    def mvnPath = '/usr/bin/mvn'  // full path to executable

    try {

        stage('Checkout') {
            checkout scm
        }

        stage('Verify Files') {
            sh 'ls -al'
        }

        stage('Build') {
            echo "Build Number: ${currentBuild.number}"
            if (currentBuild.number < 5) {
                error("Build number < 5. Exiting...")
            }
        }

        // Ensure Maven build runs to generate the .jar file
        stage('Maven Build') {
            echo "Running Maven Build to generate .jar file..."
            sh "${mvnPath} clean install"
        }

        stage('Docker Build') {
            echo "Building Docker image ${IMAGE_NAME}:${BUILD_NUMBER}..."
            sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
        }

        stage('Docker Login') {
            withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
            }
        }

        stage('Docker Push') {
            echo "Pushing Docker image ${IMAGE_NAME}:${BUILD_NUMBER}..."
            sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
        }

        stage('Test') {
            sh 'echo "MAVEN_HOME: $MAVEN_HOME"'   
            sh "which mvn"                       
            sh "$mvnPath -v"                      
            sh "$mvnPath test"                    
        }

        stage('Deploy') {
            sh """
            echo "Deploying..."
            docker run -d -p 9000:8080 --name jenkins_lab02 ${IMAGE_NAME}:${BUILD_NUMBER}
            """
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {

        echo 'Pipeline finished'
        if (currentBuild.result == 'SUCCESS') {
            echo 'Build, test, and deployment were successful!'
        } else {
            echo 'There was an error'
        }

        echo "Cleaning up Docker containers..."
        sh "docker rm -f jenkins_lab02 || true"
        sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
    }
}


pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t jenkins-cicd-demo:${BUILD_NUMBER} -t jenkins-cicd-demo:latest .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh 'docker rm -f jenkins-cicd-app || true'
                sh 'docker run -d --name jenkins-cicd-app -p 8081:80 jenkins-cicd-demo:${BUILD_NUMBER}'
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'docker ps --filter name=jenkins-cicd-app'
            }
        }
    }

    post {
        success {
            echo 'Application built and deployed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}

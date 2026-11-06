pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh 'test -f Jenkinsfile'
            }
        }

        stage('Test') {
            steps {
                sh 'echo "M2 placeholder test passed"'
            }
        }
    }

    post {
        success {
            echo 'M2 pipeline completed successfully'
        }
        failure {
            echo 'M2 pipeline failed; inspect the stage and console log'
        }
        cleanup {
            echo 'M2 pipeline cleanup complete'
        }
    }
}

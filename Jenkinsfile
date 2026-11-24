pipeline {
    agent {
        label 'python-test'
    }

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
                sh 'python --version'
                sh 'python -m pip install --dry-run --no-index --requirement requirements.txt'
                sh 'python -m pytest -v -W error::DeprecationWarning'
            }
        }
    }

    post {
        success {
            echo 'M3 pipeline completed successfully'
        }
        failure {
            echo 'M3 pipeline failed; inspect the stage and console log'
        }
        cleanup {
            echo 'M3 pipeline cleanup complete'
        }
    }
}

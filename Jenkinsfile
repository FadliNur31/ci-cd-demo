
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.65.1-noble'
            args '--ipc=host'
        }
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Environment Info') {
            steps {
                sh 'node --version'
                sh 'npm --version'
                sh 'npx playwright --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Unit Test - Jest') {
            steps {
                sh 'npm test -- --ci'
            }
        }

        stage('Build Next.js') {
            steps {
                sh 'npm run build'
            }
        }

        stage('E2E Test - Playwright') {
            steps {
                sh 'npm run test:e2e'
            }
        }
    }

    post {
        always {
            echo 'Pipeline selesai dijalankan.'
        }
        success {
            echo 'CI berhasil!'
        }
        failure {
            echo 'CI gagal. Periksa log setiap stage.'
        }
    }
}

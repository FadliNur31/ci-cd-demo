
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.65.1-noble'
            args '--shm-size=1g -e HOME=/tmp -v /var/tmp/npm-cache:/tmp/npm-cache'
        }
    }

    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = '/tmp/npm-cache'
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
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci --prefer-offline'
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


        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withEnv([
                        'SONAR_HOST_URL=http://host.docker.internal:9000'
                    ]) {
                        script {
                            // Samakan user scanner dengan pemilik workspace
                            def workspaceOwner = sh(
                                script: "stat -c '%u:%g' .",
                                returnStdout: true
                            ).trim()

                            docker.image(
                                'sonarsource/sonar-scanner-cli:12.4.0.4321_8.4.0'
                            ).inside("--user ${workspaceOwner}") {
                                sh 'sonar-scanner -Dsonar.token="$SONAR_AUTH_TOKEN"'
                            }
                        }
                    }
                }
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


pipeline {
    agent none

    environment {
        CI = 'true'
        NPM_CONFIG_CACHE = '/tmp/npm-cache'
    }

    options {
        timestamps()
        skipDefaultCheckout(true)
    }

    stages {
        stage('CI') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.65.1-noble'
                    args '--shm-size=1g -e HOME=/tmp -v /var/tmp/npm-cache:/tmp/npm-cache'
                }
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

                stage('E2E Test - Playwright') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            agent any

            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        docker run --rm \
                            --user "$(id -u):$(id -g)" \
                            -e SONAR_HOST_URL="http://host.docker.internal:9000" \
                            -e SONAR_TOKEN="$SONAR_AUTH_TOKEN" \
                            -v "$WORKSPACE:/usr/src" \
                            -w /usr/src \
                            sonarsource/sonar-scanner-cli:12.4.0.4321_8.4.0
                    '''
                }
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

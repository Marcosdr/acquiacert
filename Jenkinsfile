pipeline {
    agent { label "my127ws" }
    environment {
        COMPOSE_DOCKER_CLI_BUILD = 1
        DOCKER_BUILDKIT = 1
        MY127WS_KEY = credentials('acquiacert-my127ws-key')
        MY127WS_ENV = "pipeline"
    }
    triggers { cron(env.BRANCH_NAME == 'develop' ? 'H H(0-6) * * *' : '') }
    stages {
        stage('Setup') {
            steps {
                sh 'ws harness download'
                sh 'ws harness prepare'
                sh 'ws enable console'
                milestone(10)
            }
        }
        stage('Checks without development dependencies') {
            steps {
                sh 'ws exec composer test-production-quality'
                sh 'ws exec app composer:development_dependencies'
                milestone(20)
            }
        }
        stage('Test')  {
            parallel {
                stage('quality')    { steps { sh 'ws exec composer test-quality'    } }
                stage('helm kubeval qa')  { steps { sh 'ws helm kubeval --cleanup qa' } }
            }
        }
        stage('Build') {
            steps {
                sh 'ws enable'
                milestone(30)
            }
        }
        stage('End-to-end Test') {
            parallel {
                stage('acceptance') { steps { sh 'ws exec composer test-acceptance' } }
                stage('lighthouse') { steps { sh 'ws lighthouse' } }
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'ws test-unit'
            }
        }
    }
    post {
        always {
            sh 'ws destroy'
            cleanWs()
        }
    }
}

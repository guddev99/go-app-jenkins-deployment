pipeline {
    agent none
    stages {
        // DEV
        stage('Build Dev') {
            agent {
                label {
                    label 'dev'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                sh 'git pull'
            }
        }

        stage('Test Dev') {
            agent {
                label {
                    label 'dev'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                sh 'go test ./...'
            }
        }

        stage('Deploy Dev') {
            agent {
                label {
                    label 'dev'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                script {
                    withEnv (['JENKINS_NODE_COOKIE=do_not_kill']) {
                        sh 'go run main.go &'
                    }
                }
            }
        }


        // PROD
        stage('Build Prod') {
            agent {
                label {
                    label 'prod'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                sh 'git pull'
            }
        }

        stage('Test Prod') {
            agent {
                label {
                    label 'prod'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                sh 'go test ./...'
            }
        }

        stage('Deploy Prod') {
            agent {
                label {
                    label 'prod'
                    customWorkspace "/opt/go-app"
                }
            }
            steps {
                script {
                    withEnv (['JENKINS_NODE_COOKIE=do_not_kill']) {
                        sh 'go run main.go &'
                    }
                }
            }
        }
    }
}

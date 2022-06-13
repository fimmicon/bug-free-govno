pipeline {
    //agent { label 'master' }
    agent any
    stages {
        stage('build') {
            steps {
                sh 'python --version'
                sh '''
                     cat /etc/centos-release
                     hostname
                '''
            }
        }
    }
}

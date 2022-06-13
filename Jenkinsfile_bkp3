pipeline {
    agent { label 'master' }
    stages {
        stage('build') {
            steps {
                sh 'python --version'
                sh 'cat /etc/centos-release'
            }
        }
    }
}

pipeline {
    agent { label 'master' }
    stages {
        stage('Hello') {
            steps {
                sh 'echo Hello World'
                echo "Build result is ${currentBuild.result}"
                echo "Build number is ${currentBuild.number}"
            }
        }
    }
}

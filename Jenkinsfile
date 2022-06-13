pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                sh 'echo Hello World'
                echo "Build result is ${currentBuild.result}"
                echo "Build number is ${currentBuild.number}"
                def currentResult = currentBuild.result ?: 'SUCCESS'
		echo "Build result is ${currentBuild.result}"
            }
        }
    }
}

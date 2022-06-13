pipeline {
    agent { label 'master' }
    stages {
        stage('Hello') {
            def currentResult = currentBuild.result ?: 'SUCCESS'
            
            steps {
                sh 'echo Hello World'
                echo "Build result is ${currentBuild.result}"
                echo "Build number is ${currentBuild.number}"
		echo "Current result is ${currentResult}"
            }
        }
    }
}

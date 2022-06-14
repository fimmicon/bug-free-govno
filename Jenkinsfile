node("master") {
        stage('build') {
                echo "Value is ${MYPARAM}"
                sh 'pwd'
                sh 'ls -la'
        }

        stage('test') {
                // Get file using input step, will put it in build directory
                print "=================Please upload your property file here ====================="
                def inputFile = input message: 'Upload file', parameters: [file(name: 'global.properties')]
                // Read contents and write to workspace
                writeFile(file: 'global.properties', text: inputFile.readToString())
                // Stash it for use in a different part of the pipeline
                stash name: 'data', includes: 'global.properties'
        }

        stage('final') {
                sh 'pwd'
                sh 'ls -la'
        }
}

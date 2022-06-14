node("master") {
        stage('build') {
                sh 'pwd'
                sh 'ls -la'
        }

/*
        stage('test') {
                // Get file using input step, will put it in build directory
                print "=================Please upload your property file here ====================="
                def inputFile = input message: 'Upload file', parameters: [file(name: 'image_version.json')]
                // Read contents and write to workspace
                writeFile(file: 'image_version.json', text: inputFile.readToString())
                // Stash it for use in a different part of the pipeline
                stash name: 'data', includes: 'image_version.json'
        }
*/

        stage('final') {
                sh 'echo $FILE'
                sh 'echo $FILE | base64 -d'
                withFileParameter('FILE') {
                    sh 'cat $FILE > list'
                }
                sh 'ls -la'
                sh 'cat list'

        }
}

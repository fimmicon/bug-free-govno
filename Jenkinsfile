node("master") {

/*
        stage('build') {
                sh 'pwd'
                sh 'ls -la'
        }

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

        stage('build') {
                // The length of uploaded file is not zero
                sh '''
                   if [ -z "$IMAGE_VERSION" ];
                   then 
                         echo "=================Uploaded image_version file is empty================="
                         exit 1;
                   else
                         echo $IMAGE_VERSION | base64 -d > image_version.json
                         python -mjson.tool image_version.json &> /dev/null || ( echo "=================Uploaded image list file has invalid json syntax=================" && exit 1 )
                   fi
                '''

/*
                withFileParameter('FILE') {
                    sh 'cat $FILE '
                }
*/

                sh 'ls -la'
        }
}

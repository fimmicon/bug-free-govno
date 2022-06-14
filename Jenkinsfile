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
                sh 'echo $IMAGE_LIST'
                sh '''
                   if [ -z "$IMAGE_LIST" ];
                   then 
                         echo "=================Uploaded file is empty================="
                         exit 1;
                   else
                         echo $IMAGE_LIST | base64 -d > image_list.json
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

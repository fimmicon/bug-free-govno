node("dind") {

        stage('build') {
                sh 'hostname'
                sh 'pwd'
                sh 'ls -la'
                sh 'cat /etc/*release*'
                sh 'id'
                //sh 'sudo apk add jq'
                // sh ' python --version'
                // sh ' python3 --version'
                // sh 'jq --version'
                sh '''
                echo $IMAGE_VERSION | base64 -d > image_version.json
                cat image_version.json
                '''

                //def jsonSlurper = new JsonSlurper()
                //cfg = jsonSlurper.parse(new File(image_version.json))

                def cfg = readJSON file: 'image_version.json'
                //IMAGELIST = "${config.image_list}"
                //echo IMAGELIST
                println(cfg)
                println(cfg['image_list'])
                println(cfg.image_list[0])
                println(cfg.image_list['ui'])
                VERSION_UI = "${cfg.image_list['ui']}"
                echo VERSION_UI

		for (entry in cfg.image_list) {
                        print "========================================="
			println(entry.key)
			println(entry.value)
			
		}

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
        stage('build2') {
                // Check length of uploaded file is not zero and json syntax is correct
                checkout(scm)
                sh '''
                   if [ -z "$IMAGE_VERSION" ];
                   then 
                         echo "=================Uploaded image_version file is empty================="
                         exit 1;
                   else
                         echo $IMAGE_VERSION | base64 -d > image_version.json
                         cat image_version.json
                         python -mjson.tool image_version.json &> /dev/null || ( echo "=================Uploaded image list file has invalid json syntax=================" && exit 1 )
                   fi
                '''
                
                withFileParameter('FILE') {
                    sh 'cat $FILE '
                }
        }
*/
}

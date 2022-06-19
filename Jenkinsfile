import groovy.json.JsonOutput
import groovy.json.JsonSlurper

node("dind") {

        stage('build') {
                sh '''
                   if [ -z "$IMAGE_VERSION" ];
                   then
                         echo "=================Uploaded image_version file is empty================="
                         exit 1;
                   else
                         echo $IMAGE_VERSION | base64 -d > image_version.json
                         cat image_version.json
                   fi
                '''

                try {
                   readJSON file: 'image_version.json'
                } catch (e) {
                   echo "Caught: ${e} JSON  syntax not valid."
                   currentBuild.result = 'FAILURE'
                   sh 'exit 1'
                }

		component = "timerideR"
                println("=================================================")
		println(GetTagFromJson(component));
                Components = "ui,api,CS,BSM,ConfigAPI,QueueHandler,Dealer,MariaDB,Enricher,Correlator,Timerider,Notifier,Transformer,snmptrapd"
                str = Components.split(',')
                str.each{val -> 
                  println(GetTagFromJson(val))
		}
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

def GetTagFromJson (component) {
        component = component.toLowerCase()
	def cfg = readJSON file: 'image_version.json'
        def image_list = JsonOutput.toJson(cfg['image_list'])
        def list = new JsonSlurper().parseText( image_list.toLowerCase() )
	def tag = null;

	for (entry in list) {
                if ( entry[component] != null) {
                	tag = entry[component]
		}
        }
        if( tag != null) {
            return tag
        }
        else {
            println("Component " + component + " not found in given json file parameter")
            sh 'exit 1'
        }
}

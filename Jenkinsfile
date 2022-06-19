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

                def jsonSlurper = new JsonSlurper()
                //cfg = jsonSlurper.parse(new File(image_version.json))

                def cfg = readJSON file: 'image_version.json'
                //IMAGELIST = "${config.image_list}"
                //echo IMAGELIST
                
                println(cfg)
                println(cfg['image_list'])
                println(cfg.image_list[0])
                println(cfg.image_list['ui'])
                /OR
                VERSION_UI = "${cfg.image_list['ui']}"
                echo VERSION_UI

                
        }

}

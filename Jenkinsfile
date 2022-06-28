node("dind") {

        stage('build') {
		sh '''
			mkdir app-images
			touch 1.log 2.log 3.log 4.log 5.log 6.log 7.log 8.log
			echo 123 > 1.log
			mv *.log app-images/
		'''
		sh ' ls -la app-images/ '

		appInfraImages = "1,2,3,4,5,6,7,8,mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk".split(',')
		LIST = ''
		for (comp in appInfraImages) {
			a = "app-images/${comp}.log"
			if ( fileExists("app-images/${comp}.log") ) {
				LIST+="$a "
			}
		}
		echo "$LIST"
		sh "echo $LIST"
		withEnv(["list=${LIST}"]) {
			echo "inside"
			echo "$LIST"
			sh "echo $LIST"
		}
		
		sh 'tar -cvf app-infra-images.tar ${LIST} --remove-files'
		sh "tar-tf app-infra-images.tar"
		sh "ls -la"
		sh "ls -la app-images/"
		
       }

}

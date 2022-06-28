node("dind") {

        stage('build') {
		appInfraImages = "mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk".split(',')
		LIST = ''
		for (comp in appInfraImages) {
			if ( fileExists($comp) ) {
				echo 'Yes'
			} else {
				LIST+="$comp "
			}
		}
		echo "$LIST"
       }

}

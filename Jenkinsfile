node("dind") {

        stage('build') {
		appInfraImages = [ mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk ]
		for (comp in appInfraImages) {
		    sh "echo $comp"
		}
       }

}

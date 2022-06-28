node("dind") {

        stage('build') {
		app-infra-images = "mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk"
		for (comp in app-infra-images) {
		    sh "echo ${comp}"
		}
       }

}

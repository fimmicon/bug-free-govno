node("dind") {

        stage('build') {
		appInfraImages = "mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk".split(',')
		for (comp in appInfraImages) {
		    sh """
		    [ ! -f $comp ] && LIST+="$comp "
		    echo "\$LIST"
		    """
		}
       }

}

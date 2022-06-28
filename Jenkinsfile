node("dind") {

        stage('build') {
// 		appInfraImages = [ mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk ]
// 		for (comp in appInfraImages) {
// 		    sh "echo $comp"
// 		}
		sh """
			for i in mariadb postgresql elasticsearch influxdb redis minideb-stretch minideb-latest cp-kafka kubectl_deployer k8szk ; do [ ! -f "app-images/\${i}.tar ] && LIST+="app-images/\${i}.tar "; done
                        echo "\$LIST"
		"""
       }

}

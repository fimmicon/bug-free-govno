// import groovy.json.JsonOutput
// import groovy.json.JsonSlurper

node("dind") {

        stage('build') {
		sh '''
		for i in app-images/{mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk}.tar ; do echo "$i "; done
		'''
       }

}

// import groovy.json.JsonOutput
// import groovy.json.JsonSlurper

node("dind") {

        stage('build') {
		def app-infra-images="mariadb,postgresql,elasticsearch,influxdb,redis,minideb-stretch,minideb-latest,cp-kafka,kubectl_deployer,k8szk".split(',')
		for (comp in app-infra-images) {
		    sh "echo ${comp}"
		}
       }

}

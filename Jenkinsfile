node("dind") {

        stage('build') {
		println Artifact
		println Components
		
		str = Components.split(',');
		str.each{ val ->
			println(val)
		}
		if (Components == "") {
			println("Empty")
		}
		
		if (Components ==~ /.*Timerider.*/){ 
			println("Timerider tuta")
		}
		if (Components == "ui,api,CS,BSM,ConfigAPI,QueueHandler,Dealer,MariaDB,Enricher,Correlator,Timerider,Notifier,Transformer,snmptrapd,sqlrunner"){ 
			println("full length")
		} else {
			println(" not full length")
		}
		
       }

}

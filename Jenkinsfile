node("dind") {

        stage('build') {
		println Components
		if (Components == "") {
			println("Empty")
		}
		
		if (Components ==~ /.*UI.*/){ 
			println("UI tuta")
		}
		if (Components == "ui,api,CS,BSM,ConfigAPI,QueueHandler,Dealer,MariaDB,Enricher,Correlator,Timerider,Notifier,Transformer,snmptrapd,sqlrunner"){ 
			println("full length")
		} else {
			println(" not full length")
		}
		
       }

}

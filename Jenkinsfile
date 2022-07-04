node("dind") {

        stage('build') {
		println(Artifact)
		println(Components)
		
		Artifact = Artifact.split(',')
		if ( !(Artifact.contains('app-infra') && Components ==~ /.*MariaDB.*/  )) {
			println("=================app-infra is selected, but MariaDB not selected=================")
			return
		}

       }

}

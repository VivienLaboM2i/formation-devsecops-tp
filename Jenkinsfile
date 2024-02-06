pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"

              archive 'target/*.jar' //so that 
            }
        }  

 stage('Mutation Tests - PIT') {
  	steps {
    	sh "mvn org.pitest:pitest-maven:mutationCoverage"
  	}
    	post {
     	always {
       	pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
     	}
   	}
	}

 stage('Vulnerability Scan - Docker Trivy') {
   	steps {
        	withCredentials([string(credentialsId: 'trivy_github_token', variable: 'TOKEN')]) {
     sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh" 	 
     sh "sudo bash trivy-image-scan.sh"
       	}
   	}
 	}




    }
}

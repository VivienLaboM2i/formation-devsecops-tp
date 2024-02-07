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
        	withCredentials([string(credentialsId: 'trivy_sabrine', variable: 'TOKEN')]) {
catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
      sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh" 	 
      sh "sudo bash trivy-image-scan.sh"

}
       	}
   	}
 	}


		stage('SONAR SCAN  ') {
		steps {
				withCredentials([string(credentialsId: 'token_sonarqube', variable: 'TOKENSONAR')]) {

						withSonarQubeEnv('SonarQube'){
							sh "mvn clean verify sonar:sonar \
								-Dsonar.projectKey=tpsonarqube \
								-Dsonar.projectName='tpsonarqube' \
								-Dsonar.host.url=http://tp1.eastus.cloudapp.azure.com:9000 \
								-Dsonar.token=${TOKENSONAR}"

						}
				}
		}

	}


 stage('Docker Build and Push') {
  	steps {
    	withCredentials([string(credentialsId: 'dockerhub', variable: 'DOCKER_HUB_PASSWORD')]) {
      	sh 'sudo docker login -u sabrine24 -p $DOCKER_HUB_PASSWORD'
      	sh 'printenv'
      	sh 'sudo docker build -t sabrine24/devops-app:""$GIT_COMMIT"" .'
      	sh 'sudo docker push sabrine24/devops-app:""$GIT_COMMIT""'
    	}

  	}
	}

 stage('Deployment Kubernetes  ') {
  	steps {
    	withKubeConfig([credentialsId: 'kubeconfig']) {
           	sh "sed -i 's#replace#sabrine24/devops-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
           	sh "kubectl apply -f k8s_deployment_service.yaml"
         	}
  	}

	}







	






    }
}

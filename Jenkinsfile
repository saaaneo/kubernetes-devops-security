pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker_hub_login", url: ""]) {
          sh 'printenv'
          sh 'docker build -t aalhad/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push aalhad/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('Deploy App on k8s') {
      steps {
            sshagent(['k8s']) {
	    sh "sed -i 's#replace#aalhad/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "scp -o StrictHostKeyChecking=no k8s_deployment_service.yaml ubuntu@172.31.94.113:/home/ubuntu"
            script {
                try{
                    sh "ssh ubuntu@172.31.94.113 kubectl apply -f ."
                }catch(error){
                    sh "ssh ubuntu@172.31.94.113 kubectl create -f ."
					}
				}
			}
      
		}
	}
  }
}

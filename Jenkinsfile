pipeline {
    agent { label 'jenkins_slave_linux_1' }
    tools { 
        maven 'mvn' 
    }
    stages {
		stage ('Clone_Sources') {
			steps {
				checkout scm
			}
		}
        stage ('Build') {
		//Building and deploying to nexus
            steps {
                echo 'running mvn command'
				sh 'pwd'
				sh 'mvn clean org.jacoco:jacoco-maven-plugin:prepare-agent deploy -f docservice-templaterepo/pom.xml'
				
            }
        }
        
		stage ('Sonar Analysis') {
			steps {
			 jacoco()
             sh 'mvn sonar:sonar -Dmaven.test.failure.ignore=true -Dsonar.host.url=http://sonarqube:9000/sonarqube -Dsonar.login=bf810aa9ee8f7f5024cc1fc9aa24ded38fd44f44 -f docservice-templaterepo/pom.xml'
			}
		}
	}	
}	
	

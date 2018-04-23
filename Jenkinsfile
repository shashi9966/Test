#!/usr/bin/env groovy
node('jenkins_slave_linux_1')
{
	// determines that junit tests exist
	boolean junitsExist = true

	// determines the current build status. null means
	String error = null

	try {
         properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '4']]]);
		stage('checkout scm') {
			checkout scm
		}

		stage('prepare') {
		//Test111111  a22iaa23 aaa3
			// env.JAVA_HOME = "${tool 'JDK8'}"
			mvnHome = "${tool 'Maven3'}"
		}

		stage('maven clean') {
			// make sure that we build the entire project from scratch
			try {
				sh "${mvnHome}/bin/mvn clean -f oasp4j/darooms/pom.xml"
			} catch (err) {
				// workaround for following exception:
				// Execution failed for task ':clean'.

			}
		}

		stage('maven compile') {
		  try {
			sh "${mvnHome}/bin/mvn compile -f oasp4j/darooms/pom.xml"
			} catch (err) {
				// Compilation failed. Mark build as unstable
				error = "FAILURE"
			}

		}

		stage('maven test') {
			try {
			  sh "${mvnHome}/bin/mvn clean verify -f oasp4j/darooms/pom.xml"
			} catch (err) {
				// JUnits failed. Mark build as unstable
				error = "UNSTABLE"
				return
			}
			try {
				// archive the JUnit test execution
				step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
				
				step([$class: 'JacocoPublisher', execPattern:'**/**.exec', classPattern: '**/classes', sourcePattern: '**/src/main/java'])
	  
				// publishHTML([allowMissing: true, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: 'JUnit Execution'])
			} catch (err) {
				// this means that no JUnits were executed. We just catch the exception remeber that the build did not contain JUnits
				junitsExist = false
			}
		}
	
    	stage('maven sonarqube') {
			def projectName = env.JOB_NAME
			echo "#### ${projectName}"
			if (env.GERRIT_REFSPEC != null) {
				// if job was run via Gerrit Trigger in Jenkins, then we want to have the Sonar job description based on the changeId of Gerrit
				projectName = "_gerrit-" + GERRIT_PROJECT + "." + env.GERRIT_REFSPEC.replaceAll('/', '-')
				// remove illegal characters in changeId for proper Sonar job description
			}

			// remove illegal characters for proper Sonar job description
			projectName = projectName.replaceAll('/', '.')
			projectName = projectName.toLowerCase()

			
			echo "### ${projectName}"
			
			def sonarqubeParameters = "-Dsonar.host.url=http://sonarqube:9000/sonarqube -Dsonar.login=241130ba85e8364f7315338bd15ca13346919b68 -Dsonar.branch='${projectName}' -Dsonar.projectKey='${projectName}'"
			
			sonarqubeParameters += " -Dsonar.java.coveragePlugin=jacoco  -Dsonar.jacoco.reportPath=**/jacoco.exec"
			
			if (junitsExist)
			  // only provide junit tests if we really have something to show
				sonarqubeParameters += " -Dsonar.junit.reportsPath=target/surefire-reports"

			sh "${mvnHome}/bin/mvn sonar:sonar ${sonarqubeParameters} -f oasp4j/darooms/pom.xml"
		}
		
		stage('maven deploy') {
		  try {
			sh "${mvnHome}/bin/mvn deploy -f oasp4j/darooms/pom.xml"
			} catch (err) {
				// Deploy artifacts to the nexus failed. Mark build as unstable
				error = "FAILURE"
			}

		}
		
	} catch (err) {
		error = "FAILURE"
	} finally {
		// if no error occured so far, the 'error' will be null
		// sends email for recipient
		if (error != null) {
			currentBuild.result = error
			emailext(body: '${DEFAULT_CONTENT}', mimeType: 'text/html',
					replyTo: '$DEFAULT_REPLYTO', subject: '${DEFAULT_SUBJECT}',
					to: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
											[$class: 'RequesterRecipientProvider']]))
		}
	}
}

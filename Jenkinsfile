 
node {
  def app
  def dockerfile
  def anchorefile
	
  try {
    stage('Checkout') {
      // Clone the git repository
      checkout scm
      def path = sh returnStdout: true, script: "pwd"
      path = path.trim()
      dockerfile = path + "/Dockerfile"
      anchorefile = path + "/anchore_images"
    }
    stage('OWASP Dependency-Check Vulnerabilities ') {
    dependencyCheck additionalArguments: """
	    -s "." 
	    -f "ALL"
	    -o "./report/"
	    --prettyPrint
	    --disableYarnAudit""", odcInstallation: 'OWASP-Dependency-check'
	    dependencyCheckPublisher pattern: 'report/dependency-check-report.xml'
  }
    stage('SonarQube analysis') {
        def scannerHome = tool 'sonarqube';
        withSonarQubeEnv('sonarserver'){
            sh "${scannerHome}/bin/sonar-scanner 	      -Dsonar.projectKey=githubapp 	      -Dsonar.host.url=http://192.168.160.229:9000 	      -Dsonar.login=a6f932ce50564a7267bd08cc7f13e059ce8f190a 	      -Dsonar.sources=. 	      -Dsonar.report.export.path=sonar-report.json 	      -Dsonar.exclusions=report/* 	      -Dsonar.dependencyCheck.jsonReportPath=./report/dependency-check-report.json 	      -Dsonar.dependencyCheck.xmlReportPath=./report/dependency-check-report.xml 	      -Dsonar.dependencyCheck.htmlReportPath=./report/dependency-check-report.html"
        }
    }
    stage('Build') {
      // Build the image and push it to a staging repository
      app = docker.build("innogrid/$JOB_NAME", "--network host -f Dockerfile .")
      	
	docker.withRegistry('https://core.innogrid.duckdns.org', 'harbor') {
	app.push("$BUILD_NUMBER")
	app.push("latest")
	
      }
      sh script: "echo Build completed"
    }
    stage('Grype Image Scan') {
    	docker.withRegistry('https://core.innogrid.duckdns.org', 'harbor') {
	    sh 'grype innogrid/$JOB_NAME:latest --scope AllLayers'
	}
      }
      currentBuild.result = "SUCCESS"
  } catch (e) {
	echo "Exception=${e}"
        currentBuild.result = 'FAILURE'
	throw e
  } finally {
    stage('Cleanup') {
      sh script: "echo Clean up"
    	}	
     echo currentBuild.result
     // send slack notification
     if(currentBuild.result.equals("SUCCESS")){
	slackSend (channel: '#jenkins-notification', color: '#00FF00', message: "build success : Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
     }else{
	slackSend (channel: '#jenkins-notification', color: '#F01717', message: "build failed : Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
     }
   }
}
        

node {
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def server = Artifactory.server "artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def rtMavenQA = Artifactory.newMavenBuild()	
    def buildInfo
    
 rtMaven.tool = "maven"
 rtMavenQA.tool = "maven"
	
    stage('Clone sources') {
        git url: 'https://github.com/ayonchak89/MarsLanderApp.git'
    }

    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "maven"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment. casesetup-release
        rtMaven.deployer releaseRepo:'casesetup-release', snapshotRepo:'casesetup-snapshot', server: server
        rtMaven.resolver releaseRepo:'casesetup-release', snapshotRepo:'casesetup-snapshot', server: server
    }
 
    stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
    }

	
      stage('Static Code Analysis') {
      def sonarqubeScannerHome = tool name: 'sonarnew', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
      withSonarQubeEnv(credentialsId: 'sonarnew') {
        sh "${sonarqubeScannerHome}/bin/sonar-scanner -e -Dsonar.host.url=http://104.42.99.131:9000 -Dsonar.login=admin -Dsonar.password=admin -Dsonar.projectKey=CaseStudyAppFinal -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.language=java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java"
      
      }    
     }

    stage('Publish build info') { 
        server.publishBuildInfo buildInfo
    }

     stage('Deploy to QA') {
	deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://104.42.97.83:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
    }

    
      stage('Functional Testing') {
	//rtMaven.tool = "maven"
	//git 'https://github.com/ayonchak89/MarsLanderApp.git'
	buildInfo = rtMavenQA.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports\\', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
	      
	}
	
	/*
   	stage('Performance Testing') {
	//blazeMeterTest credentialsId: 'BlazeMeterKey', testId: '7869963.taurus', workspaceId: '461106'
	blazeMeterTest credentialsId: 'BlazeMeterKey', testId: '7886490.taurus', workspaceId: '473462'
	}
	*/
	stage('Deploy to Production') {
	deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://104.42.97.83:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
    	}

    
      stage('Sanity Testing') {
	//rtMaven.tool = "maven"
	//git 'https://github.com/ayonchak89/MarsLanderApp.git'
	buildInfo = rtMavenQA.run pom: 'Acceptancetest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports\\', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
	      
	}	
	
	stage ('Slack Confirmation') {
		slackSend channel: '#casestudydeployment', color: 'Green', message: 'Pipeline Script is Working & Deployment is Completed!', teamDomain: 'learningdevops-hq', tokenCredentialId: 'slack'
	}		
	
	stage ('JIRA Integration') {
	//jiraIssueSelector(issueSelector: [$class: 'DefaultIssueSelector'])
		
	step([$class: 'hudson.plugins.jira.JiraIssueUpdater', 
	      issueSelector: [$class: 'hudson.plugins.jira.selector.DefaultIssueSelector'],
	      scm: [$class: 'hudson.plugins.git.GitSCM'],
		labels: [ "some-label" ]]);	
		//jiraComment body: 'Issue is Fixed & deployed', issueKey: 'AYON-3'
	}
}

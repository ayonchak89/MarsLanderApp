
node {
    // Get Artifactory server instance, defined in the Artifactory Plugin administration page.
    def server = Artifactory.server "artifactory"
    // Create an Artifactory Maven instance.
    def rtMaven = Artifactory.newMavenBuild()
    def buildInfo
    
 rtMaven.tool = "maven"

    stage('Clone sources') {
        git url: 'https://github.com/ayonchak89/MarsLanderApp.git'
    }

    stage('Artifactory configuration') {
        // Tool name from Jenkins configuration
        rtMaven.tool = "maven"
        // Set Artifactory repositories for dependencies resolution and artifacts deployment.
        rtMaven.deployer releaseRepo:'Jenkins-integration', snapshotRepo:'jenkins-snapshot', server: server
        rtMaven.resolver releaseRepo:'Jenkins-integration', snapshotRepo:'jenkins-snapshot', server: server
    }
 
    stage('Maven build') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
    }

    stage('Publish build info') {
        server.publishBuildInfo buildInfo
    }

     stage('Deploy to QA') {
	deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://40.118.149.83:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
    }
    
   /*   stage('Functional Testing') {
	git 'https://github.com/ayonchak89/MarsLanderApp.git'
	buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports\\', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
	      
	}	*/
	
   	stage('Performance Testing') {
	blazeMeterTest credentialsId: 'BlazeMeterKey', testId: '7869963.taurus', workspaceId: '461106'	
	}
	 
}


node {
    def mvnHome
    stage('GitCheckout') { // for display purposes
        // Get some code from a GitHub repository
        checkout scm
    }
    stage('Build') {
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
          def pom = readMavenPom file: 'pom.xml'
          def version = pom.version.replace("-SNAPSHOT", ".${currentBuild.number}")
        
        // Run the maven build
        sh "${mvnHome}/bin/mvn -DreleaseVersion=${version} -DdevelopmentVersion=${pom.version} -DpushChanges=false -DlocalCheckout=true -DpreparationGoals=initialize release:prepare release:perform -B"
    }
    stage('Publish') {
           input 'Publish?'
           sh "git push ${pom.artifactId}-${version}"
    }
}

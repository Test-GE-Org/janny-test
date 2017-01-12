node ("mesos-java8") {
    def mvnHome
    def artServer = Artifactory.server('R2-artifactory')
    def rtMaven = Artifactory.newMavenBuild()

    stage('GitCheckout') { // for display purposes
        // Get some code from a GitHub repository
        checkout scm
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'

    }
    stage('Build') {
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' clean package -B -V -U -e -DskipTests"
         
         
    }
    
    stage('Static Code Analysis') {
        sh "'${mvnHome}/bin/mvn' checkstyle:checkstyle pmd:pmd findbugs:findbugs -B -e"
            step([$class: 'CheckStylePublisher', pattern: 'target/checkstyle-result.xml'])
            step([$class: 'FindBugsPublisher', pattern: 'target/findbugsXml.xml'])
        }

        stage('Documentation') {
            sh "'${mvnHome}/bin/mvn' javadoc:javadoc -B -e"
            step([$class: 'JavadocArchiver', javadocDir: 'target/site/apidocs', keepAll: false])
        }


        stage('Unit Tests') {
            sh "'${mvnHome}/bin/mvn' test-compile jacoco:prepare-agent surefire:test -B -e"
            step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/TEST-*.xml'])
        }
        
   stage('Code Coverage') {
            step([$class: 'JacocoPublisher', execPattern: 'target/coverage-reports/jacoco*.exec', exclusionPattern: '**/Messages.class'])
        }
        
        
  
  
        stage('Archive Artifacts') {
            step([$class: 'ArtifactArchiver', artifacts: 'target/*.jar', fingerprint: true])
            junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
        }

    
    stage("Deploy") {
        echo "deploying to production"
        sleep 10
        echo "finished deploy to production"
    }
   
}

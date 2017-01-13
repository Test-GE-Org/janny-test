node ("mesos-java8") {
    def artServer = Artifactory.server('R2-artifactory')
    def rtMaven = Artifactory.newMavenBuild()
    stage('GitCheckout') { 
        checkout scm
    }
    stage('Build') {
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer.artifactDeploymentPatterns.addInclude("target/*.jar")
        rtMaven.deployer.deployArtifacts = true
        rtMaven.tool = 'M3'
        def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
        artServer.publishBuildInfo buildInfo
    }

    stage('Unit Tests') {
        rtMaven.run pom: 'pom.xml', goals: 'test-compile jacoco:prepare-agent surefire:test -B -e'
        step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/TEST-*.xml'])
    }

    stage('Static Code Analysis') {
        rtMaven.run pom: 'pom.xml', goals: 'checkstyle:checkstyle pmd:pmd findbugs:findbugs -B -e'
        step([$class: 'CheckStylePublisher', pattern: 'target/checkstyle-result.xml'])
        step([$class: 'FindBugsPublisher', pattern: 'target/findbugsXml.xml'])
    }

    stage('Documentation') {
        rtMaven.run pom: 'pom.xml', goals: 'javadoc:javadoc -B -e'
        step([$class: 'JavadocArchiver', javadocDir: 'target/site/apidocs', keepAll: false])
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

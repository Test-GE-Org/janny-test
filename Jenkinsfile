#!/usr/bin/env groovy

try 
{
    node ("mesos-java8"){
        def artServer = Artifactory.server('R2-artifactory')
        def branchName = env.BRANCH_NAME

        def rtMaven = Artifactory.newMavenBuild()
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer.artifactDeploymentPatterns.addInclude("target/*.jar")
        rtMaven.deployer.deployArtifacts = true
        rtMaven.tool = 'M3'

        stage('GitCheckout') { 
            echo branchName
            checkout scm
        }

        stage('Build') {
            // Most typical, if you're not cloning into a sub directory
            gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            // short SHA, possibly better for chat notifications, etc.
            def shortCommit = gitCommit.take(6)
            echo shortCommit

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
        stage("Deploy To Dev") {
            echo "deploying to dev"
            sleep 10
            echo "finished deploy to dev"
        }

        if(branchName == "master"){
            promoteToStaging();
            waitForApproval();
            promoteToProduction();
        }

    }
  
}
catch (exc) {
    echo "Caught: ${exc}"
}


def promoteToStaging(){
        stage("Promote to stage") {
            echo "deploying to stage"
            sleep 10
            echo "finished deploy to stage"
        }
        stage("Integration test") {
            echo "integration test"
            sleep 10
            echo "integration test"
        }
        stage("Compliance test") {
            echo "Compliance test"
            sleep 10
            echo "Compliance test"
        }
}
def waitForApproval(){
     stage("Ready to go production?") {
            echo "Wait for input"
            sleep 10
            echo "input recieved"
        }
}

def promoteToProduction(){
     stage("Promote to production") {
            echo "deploying to production"
            sleep 10
            echo "finished deploy to production"
        }
        stage("Acceptance test") {
            echo "Acceptance test"
            sleep 10
            echo "Acceptance test"
        }
}




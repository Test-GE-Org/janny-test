#!/usr/bin/env groovy

try 
{
    node ("mesos-java8"){
        def artServer = Artifactory.server('R2-artifactory')
        def branchName = env.BRANCH_NAME
        def shortCommit 
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
            shortCommit = gitCommit.take(6)
            echo shortCommit

            def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
            artServer.publishBuildInfo buildInfo
            stash includes: 'target/*.jar', name: 'artifact'
            stash includes: 'manifest.yml', name: 'manifest'
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
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                api_url = "deployer-api-devops-dev.run.aws-usw02-pr.ice.predix.io"
                domain_url = "run.aws-usw02-pr.ice.predix.io"
                metastore_url = "metastore-devops-dev.run.aws-usw02-pr.ice.predix.io"
                org ="predix-devops"
                space = "dev"
                user_name = "pd-stg-admin"
                token_id = "i7GBivOW3zTyfzIhc6PAZHxL"


                unstash 'artifact'
                unstash 'manifest'
                pom = readMavenPom file: 'pom.xml'
                artifact_url = "*.jar"
                manifest_url ="*.yml"
                build_number = "${env.BUILD_NUMBER}"
                app_id = "${pom.artifactId}"
                version = "${pom.version}"
                app_name = "${pom.artifactId}"
                echo "Authenticating for deploy"
                sh 'pcd deploy auth -a ${api_url} -d ${domain_url} -m${metastore_url} -o ${org} -s${space} -u ${user_name} -tid ${token_id}'
                echo "Authentication done"
                echo "Deploying the artifacts"
                sh 'pcd deploy -ar ${artifact_url} -m ${manifest_url} -b ${build_number} -id ${app_id} -v ${version} -n ${app_name}' 
                echo "Deployed the artifacts"
            }
            else{
                echo "PCD tool not found"
            }
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





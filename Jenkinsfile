#!/usr/bin/env groovy

try 
{
    node ("predixci-jdk-1.8"){
        def artServer = Artifactory.server('R2-artifactory')
        def branchName = env.BRANCH_NAME
        def shortCommit 
        def rtMaven = Artifactory.newMavenBuild()
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer server: artServer, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
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
            stash includes: 'pom.xml', name:'pom'
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

    }
    node ("predixci-pcd"){
        stage("Deploy To Dev") {
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                api_url = "<deployer_api_url_goes_here>"
                domain_url = "<domain_url_goes_here>"
                metastore_url = "<metastore_url_goes_here>"
                org ="<org_goes_here>"
                space = "<space_goes_here>"
                user_name = "<User_id_goes_here>"
                token_id = "<token_goes_here>"


                unstash 'artifact'
                unstash 'manifest'
                unstash 'pom'
                pom = readMavenPom file: 'pom.xml'
                artifact_url = "<artifact_url_goes_here>"
                manifest_url ="<manifest_url_goes_here>"
                build_number = "${env.BUILD_NUMBER}"
                app_id = "${pom.artifactId}"
                version = "${pom.version}"
                app_name = "${pom.artifactId}"
                deploy(api_url,domain_url,metastore_url,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
            }
            else{
                echo "PCD tool not found"
            }
        }
    }
    
    echo "${branchName}"
    echo "master".equals(branchName)
    echo "master".equals("${branchName}")
    if("master".equals(branchName)){
        waitForApprovalStaging();
        promoteToStaging();
        waitForApprovalProduction();
        promoteToProduction();
    }

    
  
}
catch (exc) {
    echo "Caught: ${exc}"
}

void deploy(String api_url,String domain_url,String metastore_url,String org,String space,String user_name,String token_id,String artifact_url,String manifest_url,String build_number,String app_id,String version,String app_name){
        echo "Authenticating for deploy another ${api_url}"
        sh "pcd deploy auth -a '${api_url}' -d '${domain_url}' -m '${metastore_url}' -o '${org}' -s '${space}' -u '${user_name}' -tid '${token_id}'"
        echo "Authentication done"
        echo "Deploying the artifacts"
        sh "pcd deploy -ar '${artifact_url}' -m '${manifest_url}' -b '${build_number}' -id '${app_id}' -v '${version}' -n '${app_name}'"
        echo "Deployed the artifacts"
}


def promoteToStaging(){
    node ("predixci-pcd"){
        stage("Promote to stage") {
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                api_url = "<deployer_api_url_goes_here>"
                domain_url = "<domain_url_goes_here>"
                metastore_url = "<metastore_url_goes_here>"
                org ="<org_goes_here>"
                space = "<space_goes_here>"
                user_name = "<User_id_goes_here>"
                token_id = "<token_goes_here>"

                unstash 'artifact'
                unstash 'manifest'
                unstash 'pom'
                pom = readMavenPom file: 'pom.xml'
                artifact_url = "<artifact_url_goes_here>"
                manifest_url ="<manifest_url_goes_here>"
                build_number = "${env.BUILD_NUMBER}"
                app_id = "${pom.artifactId}"
                version = "${pom.version}"
                app_name = "${pom.artifactId}"
                deploy(api_url,domain_url,metastore_url,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
            }
            else{
                echo "PCD tool not found"
            }
        }
    }

    node ("predixci-jdk-1.8"){
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
}
def waitForApprovalStaging(){
    stage("Ready to go staging?") {
        timeout(time:1, unit:'DAYS') {
            input message:'Approve deployment to staging?', submitter: 'it-ops'
        }
    }
}

def waitForApprovalProduction(){
    stage("Ready to go production?") {
        timeout(time:5, unit:'DAYS') {
            input message:'Approve deployment to production?', submitter: 'it-ops'
        }
    }
}


def promoteToProduction(){
    node ("predixci-pcd"){
        stage("Promote to production") {
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                api_url = "<deployer_api_url_goes_here>"
                domain_url = "<domain_url_goes_here>"
                metastore_url = "<metastore_url_goes_here>"
                org ="<org_goes_here>"
                space = "<space_goes_here>"
                user_name = "<User_id_goes_here>"
                token_id = "<token_goes_here>"


                unstash 'artifact'
                unstash 'manifest'
                unstash 'pom'
                pom = readMavenPom file: 'pom.xml'
                artifact_url = "<artifact_url_goes_here>"
                manifest_url ="<manifest_url_goes_here>"
                build_number = "${env.BUILD_NUMBER}"
                app_id = "${pom.artifactId}"
                version = "${pom.version}"
                app_name = "${pom.artifactId}"
                deploy(api_url,domain_url,metastore_url,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
            }
            else{
                echo "PCD tool not found"
            }
        }
    }
    node ("predixci-jdk-1.8"){
        stage("Acceptance test") {
            echo "Acceptance test"
            sleep 10
            echo "Acceptance test"
        }
    }
}



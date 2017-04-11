#!/usr/bin/env groovy
def branchName = env.BRANCH_NAME
def complianceEnabled = true;

// Tinfoil WebApp Scan variables
def TF_webappURL = "https://tinfoil.ice.ge.com/api/v1/sites/ci-cd-demo/scans"
def TF_appURL = "https://cicd-demo-host.run.aws-usw02-pr.ice.predix.io/greeting"
def TF_siteName = "ci-cd-demo"
def TF_webappToken = "S9T/8ODFvFMurjxO9/6Fz0BE"
def TF_webappAccessKey = "/Hx9SNDwi8DxYwjcVrmzLANM"

// Tinfoil API Scan variables. apiID = API_ID, not API_Collection_ID
def TF_apiURL = "https://tf01-api.tf.ice.predix.io"
def TF_apiID = 151
def TF_apiToken = "PV0fhF7AbvdrcdAiyA8m9Fww"
def TF_apiAccessKey = "B-zow6i9PsjGe85-fzMMjqdo"

try 
{
    node ("predixci-jdk-1.8"){
        
        sh 'ls -lrt  /usr/local/bin/'
        
/*
        def artServer = Artifactory.server('R2-artifactory')
        def shortCommit 
        def rtMaven = Artifactory.newMavenBuild()
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer server: artServer, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
*/ //        rtMaven.deployer.artifactDeploymentPatterns.addInclude("**/target/*.jar")
/*        rtMaven.deployer.deployArtifacts = true
        rtMaven.tool = 'M3'
*/
        stage('GitCheckout') { 
            echo branchName
            checkout scm
        }
/*
        stage('Build') {

            // Most typical, if you're not cloning into a sub directory
            gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            // short SHA, possibly better for chat notifications, etc.
            shortCommit = gitCommit.take(6)
            echo shortCommit

             def buildInfo = rtMaven.run pom: 'pom.xml', goals: '-B -DskipTests clean install'
            artServer.publishBuildInfo buildInfo
*/

//            try{
//                stash includes: '**/target/*.jar', name: 'artifact'
//            }catch(Exception e){
//                echo "No Jar files to stash"
//            }
//            try{
//                stash includes: '**/manifest.yml', name: 'manifest'
//            }catch(Exception e){
//                echo "No manifest file to stash"
//            }
//            stash includes: 'pom.xml', name:'pom'
//            
//        }
        
 //       stage('Unit Tests') {
//            //rtMaven.run pom: 'pom.xml', goals: 'test-compile jacoco:prepare-agent surefire:test -B -e'
 //           rtMaven.run pom: 'pom.xml', goals: '-B clean test -Dmaven.test.failure.ignore'
//            step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
//        }
        
   //      stage('Static Code Analysis') {
   //         rtMaven.run pom: 'pom.xml', goals: 'checkstyle:checkstyle pmd:pmd findbugs:findbugs -B -e'
   //         step([$class: 'CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
   //         step([$class: 'FindBugsPublisher', pattern: '**/target/findbugsXml.xml'])
   //     }


//        stage('Archive Artifacts') {
//            //step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
//            archive '**/target/*.jar'
//        }
      
//        stage('Deploy') {
//            //unstash is necessary to use the artifact stored in 'Build' stage
//            unstash 'artifact'
            
//            // Do not change the credentialsId.  This is the credential to deploy to predix-devops-runtime on CF1
//            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '2ff06a29-9500-4251-bd7a-903f54eab497', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
//               sh 'echo uname=$USERNAME pwd=$PASSWORD'
//               sh 'cf login -a https://api.system.aws-usw02-pr.ice.predix.io -u $USERNAME -p $PASSWORD -o predix-devops-runtime -s dev'
//            }
            
//            sh 'cf push'
//            sh 'cf a'
//        }
        

        stage('TinFoil-WebappScan') {
            echo "Calling Tinfoil WebApps Scan"  
            
            // Command line with no parameters
            // sh '/usr/bin/curl --insecure -v https://tinfoil.ice.ge.com/api/v1/sites/ci-cd-demo/scans -X POST -d "site[name]=ci-cd-demo" -d "site[url]=https://cicd-demo-host.run.aws-usw02-pr.ice.predix.io/greeting" -H "Authorization:Token token=S9T/8ODFvFMurjxO9/6Fz0BE, access_key=/Hx9SNDwi8DxYwjcVrmzLANM"'
            
            // Command line with parameters
            // sh '/usr/bin/curl --insecure -v '+ TF_webappURL + ' -X POST -d "site[name]=' + TF_siteName +'" -d "site[url]=' + TF_appURL +'" -H "Authorization:Token token=' + TF_webappToken +', access_key=' + TF_webappAccessKey +'"'
            // sh "/usr/bin/curl --insecure -v ${TF_webappURL} -X POST -d 'site[name]=${TF_siteName}' -d 'site[url]=${TF_appURL}' -H 'Authorization:Token token=${TF_webappToken}, access_key=${TF_webappAccessKey}'"
            
            // Shell scripts with parameters
            // sh 'tinfoil_startscan_wparams.sh '+ TF_webappURL + ' '+ TF_siteName + ' '+ TF_appURL + ' '+ TF_webappToken + ' '+ TF_webappAccessKey +''
            sh "tinfoil_startscan_wparams.sh ${TF_webappURL} ${TF_siteName} ${TF_appURL} ${TF_webappToken} ${TF_webappAccessKey}"
        }
        
        stage('TinFoil-APIScan') {
            echo "Calling Tinfoil API Scan"
            // temporary called it without .sh
            sh "tinfoil_startscan_checkstatus ${TF_apiToken} ${TF_apiAccessKey} ${TF_apiURL} ${TF_apiID}"
            // sh "tinfoil_startscan_checkstatus.sh ${TF_apiToken} ${TF_apiAccessKey} ${TF_apiURL} ${TF_apiID}"
        }
        
    }

/*
    if(complianceEnabled){
            echo "Compliance Enabled. Triggering white source scanning."
            doWhiteSourceScan();
    }


    node ("predixci-pcd"){
        stage("Deploy To Dev") {
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                env_val = "cf1"
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
                deploy(env_val,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
            }
            else{
                echo "PCD tool not found"
            }
        }
    }
    
    if("master".equals(branchName)){
        echo "Commit on master branch. Promoting to staging"
        waitForApprovalStaging();
        promoteToStaging();
        waitForApprovalProduction();
        promoteToProduction();
    } 
*/

}
catch (exc) {
    echo "Caught: ${exc}"
}

/*
def doWhiteSourceScan(){
    node ("predixci-whitesource"){
        def artServer = Artifactory.server('R2-artifactory')
        def shortCommit 
        def rtMaven = Artifactory.newMavenBuild()
        rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
        rtMaven.deployer server: artServer, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
        rtMaven.deployer.artifactDeploymentPatterns.addInclude("target/*.jar")
        rtMaven.deployer.deployArtifacts = true
        rtMaven.tool = 'M3'


        stage("White Source Compliance Scan"){
            checkout scm
            gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            shortCommit = gitCommit.take(6)
            def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'dependency:copy-dependencies'
            pom = readMavenPom file: 'pom.xml'
            app_name = "${pom.artifactId}"
            sh 'cp /var/local/*.config ./'
            sh "sed -ie 's/<project_name>/${app_name}/g' whitesource-fs-agent.config"
            sh 'java -jar /var/local/whitesource-fs-agent-1.7.2.jar -d target/dependencies'
        }
    }   
}

def promoteToStaging(){
    node ("predixci-pcd"){
        stage("Promote to stage") {
            pcdOutput = sh(returnStatus: true, script: 'pcd')
            if(pcdOutput == 0)
            {
                echo "PCD tool is available"
                env_val = "cf1"
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
                deploy(env_val,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
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
                env_val = "cf1"
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
                deploy(env_val,org,space,user_name,token_id,artifact_url,manifest_url,build_number,app_id,version,app_name);
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

void deploy(String envVal,String org,String space,String user_name,String token_id,String artifact_url,String manifest_url,String build_number,String app_id,String version,String app_name){
        sh "pcd deploy auth -env '${envVal}' -o '${org}' -s '${space}' -u '${user_name}' -tid '${token_id}'"
        echo "Authentication done"
        echo "Deploying the artifacts"
        sh "pcd deploy -a '${artifact_url}' -m '${manifest_url}' -b '${build_number}' -id '${app_id}' -v '${version}' -n '${app_name}'"
        echo "Deployed the artifacts"
}
*/


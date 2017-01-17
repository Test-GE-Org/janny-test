#!/usr/bin/env groovy


try {
        node ("mesos-java8") {
            def artServer = Artifactory.server('R2-artifactory')
            def rtMaven = Artifactory.newMavenBuild()
            rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
            rtMaven.deployer.artifactDeploymentPatterns.addInclude("target/*.jar")
            rtMaven.deployer.deployArtifacts = true
            rtMaven.tool = 'M3'

            stage('GitCheckout') { 
                checkout scm: [$class: 'GitSCM' ,branch: '  ']
            }


            stage('Set Version') {
                def originalV = version();
                def major = originalV[1];
                def minor = originalV[2];
                def patch  = Integer.parseInt(originalV[3]) + 1;
                def v = "${major}.${minor}.${patch}"
                if (v) {
                    echo "Building version ${v}"
                }
                rtMaven.run pom: 'pom.xml', goals: '-B versions:set -DgenerateBackupPoms=false -DnewVersion=${v}'      
                sh 'git add .'
                sh "git commit -m 'Raise version'"
                sh "git tag v${v}"
            }

            stage('Print info') {
               // Most typical, if you're not cloning into a sub directory
                gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                // short SHA, possibly better for chat notifications, etc.
                shortCommit = gitCommit.take(6)

                echo $shortCommit


                def causes = currentBuild.rawBuild.getCauses()


                echo $causes
            }


        }
      
}
catch (exc) {
    echo "Caught: ${exc}"

    String recipient = 'project-admin@ge.com'

    /*mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) failed",
            body: "It appears that ${env.BUILD_URL} is failing, somebody should do something about that",
              to: recipient,
         replyTo: recipient,
            from: 'predix-cicd@build.ge.com'*/
}

def version() {
    def matcher = readFile('pom.xml') =~ '<version>(\\d*)\\.(\\d*)\\.(\\d*)(-SNAPSHOT)*</version>'
    matcher ? matcher[0] : null
}




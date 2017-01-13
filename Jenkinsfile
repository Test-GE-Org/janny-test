try {
    stage('Dev') {
        node ("mesos-java8") {
            def artServer = Artifactory.server('R2-artifactory')
            def rtMaven = Artifactory.newMavenBuild()
            rtMaven.resolver server: artServer, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
            rtMaven.deployer.artifactDeploymentPatterns.addInclude("target/*.jar")
            rtMaven.deployer.deployArtifacts = true
            rtMaven.tool = 'M3'

            stage('GitCheckout') { 
                checkout scm: [branch: 'develop']
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



            stage('Build') {
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
                echo "deploying to dev"
                sleep 10
                echo "finished deploy to dev"
            }
        }
      
    }

    stage('Staging') {
        node ("mesos-java8") {
            def artServer = Artifactory.server('R2-artifactory')
            def rtMaven = Artifactory.newMavenBuild()
            stage('GitCheckout') { 
                checkout scm: [branch: 'master']
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



            stage('Build') {
                def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
                artServer.publishBuildInfo buildInfo
            }




            stage('Archive Artifacts') {
                step([$class: 'ArtifactArchiver', artifacts: 'target/*.jar', fingerprint: true])
                junit '**/target/surefire-reports/TEST-*.xml'
                archive 'target/*.jar'
            }

            stage("Deploy") {
                echo "deploying to staging"
                sleep 10
                echo "finished deploy to staging"
            }
        }
    }



    stage('Production') {
        node ("mesos-java8") {
            stage('Get staging artifacts') { 
                echo "Retrieve staging build from artifactory"
                sleep 10
                echo "Retrieved staging build from artifactory"
            }
           
            stage("Deploy") {
                echo "deploying to production"
                sleep 10
                echo "finished deploy to production"
            }
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




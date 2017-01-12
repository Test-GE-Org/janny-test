node ("mesos-java8") {
    def mvnHome
    stage('GitCheckout') { // for display purposes
        // Get some code from a GitHub repository
        checkout scm
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
    }
    stage('Build') {
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' clean package -B -V -U -e -DskipTests"
         
         
    }
    
    stage('Static Code Analysis') {
        sh "'${mvnHome}/bin/mvn' checkstyle:checkstyle pmd:pmd findbugs:findbugs -B -e"
            step([$class: 'CheckStylePublisher', pattern: 'target/checkstyle-result.xml'])
            step([$class: 'FindBugsPublisher', pattern: 'target/findbugsXml.xml'])
            step([$class: 'PmdPublisher', pattern: 'target/pmd.xml'])
            step([$class: 'DryPublisher', pattern: 'target/cpd.xml'])
            step([$class: 'TasksPublisher', high: 'FIXME', low: '', normal: 'TODO', pattern: 'src/**/*.java'])
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
            step([$class: 'JacocoPublisher', execPattern: 'target/jacoco.exec', exclusionPattern: '**/Messages.class'])
        }
        
        
        
    def testingJobs = [:]

    testingJobs["Integration testing"] = {
        node {
            echo "deploying to staging environment."
            sleep 10
            echo "deployed staging sucessfully"
        }
    }
    testingJobs["Performance testing"] = {
        node {
            echo "deploying to perf environment."
            sleep 10
            echo "deployed sucessfully"
        }
    }
    
    
    stage('Testing') {
        parallel testingJobs
    }
  
  
  
  
  
    
    def complTests = [:]
    complTests["OSSCAR"] = {
        node {
            echo "running OSSCAR tests"
            sleep 10
            echo "finished OSSCAR tests"
        }
    }
    complTests["Tinfoil"] = {
        node {
            echo "running tinfoil tests"
            sleep 10
            echo "finished tinfoil tests"
        }
    }
    stage("ComplianceTests") {
        echo "starting compliance tests.."
        parallel complTests
        echo "finished compliance tests"
    }


        stage('Archive Artifacts') {
            step([$class: 'ArtifactArchiver', artifacts: 'target/*.hpi,target/*.jpi', fingerprint: true])
        }


    stage('StageArtifacts') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
    }
    
    stage("Deploy") {
        echo "deploying to production"
        sleep 10
        echo "finished deploy to production"
    }
   
}

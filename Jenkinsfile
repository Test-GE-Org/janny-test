node {
    def mvnHome
    stage('GitCheckout') { // for display purposes
        // Get some code from a GitHub repository
        checkout scm
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.
        mvnHome = tool 'M3'
    }
     stage('Unit test') {
    
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' test"
    }
    stage('Build') {
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
    }
  
    stage('Jacoco Code coverage') {
        echo "deploying to dev env"
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

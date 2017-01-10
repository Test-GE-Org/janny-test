node ("mesos-java8"){
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
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
    }
    stage('DeployToDev') {
        echo "deploying to dev env"
    }
    stage('StageArtifacts') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
    }
    stage('PromoteStaging') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archive 'target/*.jar'
    }
    def deployJobs = [:]

    deployJobs["staging"] = {
        node {
            echo "deploying to staging environment."
            sleep 10
            echo "deployed staging sucessfully"
        }
    }
    deployJobs["perf"] = {
        node {
            echo "deploying to perf environment."
            sleep 10
            echo "deployed sucessfully"
        }
    }
    stage('Deploy') {
        parallel deployJobs
    }
    def UItestJobs = [:]
    UItestJobs["chrome"] = {
        node {
            echo 'Starting Selenium Tests For Chrome Browser'
            sleep 10
            echo 'Finished Chrome tests'
        }
    }
    UItestJobs["safari"] = {

        node {
            echo 'Starting Selenium Tests For safari Browser'
            sleep 10
            echo 'Finished firefox tests'
        }
    }
    UItestJobs["firefox"] = {

        node {
            echo 'Starting Selenium Tests For safari Browser'
            sleep 10
            echo 'Finished firefox tests'
        }
    }
    UItestJobs["IE"] = {

        node {
            echo 'Starting Selenium Tests For safari Browser'
            sleep 10
            echo 'Finished firefox tests'
        }
    }
    stage('IntegrationTests') {
        echo "running integration tests"
        sleep 10
        echo "finished integration tests"
    }
    stage("SeleniumTests") {
        echo "running selenium tests.."
        parallel UItestJobs
        echo "finished selenium tests"
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

    stage("Promote") {
        echo "running approval stage"
        echo "waiting for approval to promote production"
        sleep 10
        echo "got approval to promote production..proceeding"
    }
    stage("Deploy") {
        echo "deploying to production"
        sleep 10
        echo "finished deploy to production"
    }
    stage("SmokeTest") {
        echo "Running smoke tests on production"
	sh "date"
        sleep 10
        echo "smoke tests completed. PASSED"
    }
    stage("Finish") {
        echo "Switch Routes, new version live"
        sleep 10
        echo "routes switched. Production OK"
    }
}

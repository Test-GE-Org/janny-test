@Library('servers') import demo.Servers

jettyUrl = 'http://localhost:8081/'
servers = new Servers(this)

stage('Dev') {
    node {
        checkout scm
        mvn '-o clean package'
        dir('target') {stash name: 'war', includes: 'x.war'}
    }
}

stage('QA') {
    parallel(longerTests: {
        runTests(30)
    }, quickerTests: {
        runTests(20)
    })
    echo "Test results: ${testResult(currentBuild)}"
}

milestone 1
stage('Staging') {
    lock(resource: 'staging-server', inversePrecedence: true) {
        milestone 2
        node {
            servers.deploy 'staging'
        }
        input message: "Does ${jettyUrl}staging/ look good?"
    }
    try {
        checkpoint('Before production')
    } catch (NoSuchMethodError _) {
        echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
    }
}

milestone 3
stage ('Production') {
    lock(resource: 'production-server', inversePrecedence: true) {
        node {
            sh "wget -O - -S ${jettyUrl}staging/"
            echo 'Production server looks to be alive'
            servers.deploy 'production'
            echo "Deployed to ${jettyUrl}production/"
        }
    }
}
def mvn(args) {
    sh "${tool 'Maven 3.x'}/bin/mvn ${args}"
}

def runTests(duration) {
    node {
        checkout scm
        servers.withDeployment {id ->
            mvn "-o -f sometests test -Durl=${jettyUrl}${id}/ -Dduration=${duration}"
        }
        junit '**/target/surefire-reports/TEST-*.xml'
    }
}










jettyUrl = 'http://jenkins.gerritcentral.com:8081/'

def servers

/*
stage('Dev') {
    node {
	build(job: 'dev-job')
    }
}
*/

stage('QA') {
    parallel(longerTests: {
        runTests(servers, 30)
    }, quickerTests: {
        runTests(servers, 20)
    })
}

milestone 1
stage('Staging') {
    lock(resource: 'staging-server', inversePrecedence: true) {
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

milestone 2
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

def runTests(servers, duration) {
    node {
	build(job: 'dev-run-tests', parameters: [ new hudson.model.StringParameterValue('duration', "$duration") ] )
    }
}

def milestone(n) {
    try {
        steps.milestone(n)
    } catch (NoSuchMethodError _) {
        echo 'JENKINS-27039: milestone step not yet released'
    }
}

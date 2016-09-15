jettyUrl = 'http://jenkins.gerritcentral.com:8081/'

def servers

stage('Dev') {
    node {
	build(job: 'dev-build')
    }
}

stage('QA') {
    runTests()
}

milestone 1
stage('Staging') {
    lock(resource: 'staging-server', inversePrecedence: true) {
        node {
            deploy()
        }
        input message: "Does ${jettyUrl}staging/ look good?"
    }
    try {
        checkpoint('Before production')
    } catch (NoSuchMethodError _) {
        echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
    }
}

def runTests() {
    node {
	build(job: 'dev-run-tests')
    }
}

def deploy() {
    node {
	build(job: 'deploy-to-staging')
    }
}

def milestone(n) {
    try {
        steps.milestone(n)
    } catch (NoSuchMethodError _) {
        echo 'JENKINS-27039: milestone step not yet released'
    }
}

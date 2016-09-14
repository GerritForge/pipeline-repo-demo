jettyUrl = 'http://jenkins.gerritcentral.com:8081/'

def servers

stage('Dev') {
    node {
        checkout scm
        servers = load 'servers.groovy'
        mvn 'clean package'
        dir('target') {stash name: 'war', includes: 'x.war'}
    }
}

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
def mvn(args) {
    sh "${tool 'Maven 3.x'}/bin/mvn ${args}"
}

def runTests(servers, duration) {
    node {
        checkout scm
        servers.runWithServer {id ->
            mvn "-f sometests test -Durl=${jettyUrl}${id}/ -Dduration=${duration}"
        }
        junit '**/target/surefire-reports/TEST-*.xml'
    }
}

def milestone(n) {
    try {
        steps.milestone(n)
    } catch (NoSuchMethodError _) {
        echo 'JENKINS-27039: milestone step not yet released'
    }
}

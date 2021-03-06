def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t jervelocity/dcos:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'jervelocity',
            passwordVariable: 'password',
            usernameVariable: 'jervelocity'
        ]]
    ) {
        sh "docker login -u jervelocity -p password -e jervelocity@gmail.com"
        sh "docker push jervelocity/dcos:${gitCommit()}"
    }

    // Deploy
    stage 'Deploy'
    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: false,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appId: 'jer-nginx',
        docker: "jervelocity/dcos:${gitCommit()}".toString()
    )
}

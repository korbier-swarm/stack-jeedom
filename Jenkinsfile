properties(
    [
        parameters([
            booleanParam(defaultValue: false, description: 'Deploy stack ?', name: 'deployStack')
        ]),
        
        buildDiscarder( 
            logRotator( daysToKeepStr: '5', numToKeepStr: '5' ) 
        ),
        
        pipelineTriggers([
            cron('H 23 * * *')
        ])
        
    ]
)

def dockerImageUser     = 'korbier'
def dockerImagePrefix   = 'rpi' 
def dockerImageName     = 'jeedom'
def dockerImageFullName = dockerImageUser +'/' + dockerImagePrefix + '-' + dockerImageName + ':$BUILD_NUMBER'

def remote = [:]
remote.name = "perceval"
remote.host = "perceval"
remote.allowAnyHosts = true

node {
    //checkout scm
    git url: 'https://github.com/jeedom/core'
}

node {

    stage ('Build docker image') {
        dockerImage = docker.build dockerImageFullName
    }

    docker.withRegistry('', 'dockerhub') {
        stage ('Pushing docker image') {
            dockerImage.push()
            dockerImage.push('latest')
        }
    }

    stage ('Cleanup local docker image') {
        sh 'docker rmi ' + dockerImageName
    }

}


if ( params.deployToGithub ) {

    withCredentials([usernamePassword(credentialsId: 'ssh-perceval', passwordVariable: 'pwdVariable', usernameVariable: 'userVariable')]) {
        
        remote.user = userVariable
        remote.password = pwdVariable

        stage("Deploy to swarm") {
            sshPut remote: remote, from: 'docker-compose.yml', into: '/tmp'
            sshCommand remote: remote, command: 'docker stack deploy -c /tmp/docker-compose.yml home-docs'            
        }

    }

}
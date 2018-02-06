

def workdir = "dir1"

node(){
    stage('test'){
        sh "export"
        dir(workdir) {
            deleteDir()
        }
    }
    stage('get source') {
	dir(workdir) {
		git branch: 'rkudryashov', credentialsId: '243eef37-2cc2-4340-9b63-2e9b8d13182e', url: 'git@gitlab.com:nikolyabb/epam-devops-3rd-stream.git'
		sh returnStdout: true, script: 'ls -lah'
                }
	}	
    stage('run tests') {

    }
    stage('build package') {

    }
    stage('save artifact') {

    }
    stage('deploy to env') {

    }
    stage('provision env') {

    }
    stage('integration test') {

    }
    stage('send report') {

    }
}

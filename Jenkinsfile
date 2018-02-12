

def workdir = "dir1"
def art = "art"
def proc = "message-processor"
node(){
    stage('test'){
        sh "export"
        dir(workdir) {
            deleteDir()
        }
    }
    stage('get source') {
	dir(workdir) {
		git branch: 'rkudryashov', credentialsId: '650ea460-204d-4861-963b-af80d47367b0', url: 'git@gitlab.com:nikolyabb/epam-devops-3rd-stream.git'
		sh 'ls -lah'
                }
	}	
    stage('run tests') {
	dir(workdir) {
		withMaven(maven: 'maven'){
		   sh 'mvn clean test'
	    }
	}

    }
    stage('build package') {
	 dir(workdir) {
                withMaven(maven: 'maven'){
                   sh 'mvn -X clean package -Dmaven.test.skip=true'
        	}
		sh 'find . -type f -regex ".*\\.\\(jar\\|war\\)"'
			
		docker.withTool('docker'){
	    	   withDockerServer([uri: 'unix:///var/run/docker.sock']) {
			sh 'docker ps -a'
			}
		   }
		}
    }
   
    stage('save artifact') {
	 dir (workdir){
	    sh 'find ./$proc/ ! -regex "\\(.*etc/*.*\\|.*\\.jar\\)" -delete'
	    dir (proc){
		sh 'echo $\'FROM java:8\\n\\n\\nCOPY    . /workdir/\\nWORKDIR /workdir/\\nENTRYPOINT ["java"]\\nCMD ["-jar","message-processor-1.0-SNAPSHOT.jar","etc/config.properties"]\' > Dockerfile'
		}	

          }
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

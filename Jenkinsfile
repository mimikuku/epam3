

def workdir = "dir1"
def art = "art"
def proc = "docker_image_proc"
def gateway = "docker_image_gateway"
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
	    //sh 'find ./message-processor/ ! -regex "\\(.*etc/*.*\\|.*\\.jar\\)" -delete'
	    dir (proc){
		sh 'cp $(find $JENKINS_HOME/workspace/$JOB_NAME/dir1/message-processor/ -name "message-processor-1.0-SNAPSHOT.jar") .'
		sh 'cp $(find $JENKINS_HOME/workspace/$JOB_NAME/dir1/message-processor/ -name "config.properties") .' 	
		sh 'echo \'FROM java:8\\n\\n\\nCOPY . /workdir/\\nWORKDIR /workdir/\\nENTRYPOINT ["java"]\\nCMD ["-jar","message-processor-1.0-SNAPSHOT.jar","config.properties"]\' > Dockerfile'
		docker.withTool('docker'){
			withDockerRegistry([credentialsId: 'f9b46bc9-8260-4db8-821c-b9fa96fdb4f2', url: 'https://index.docker.io/v1/']) {
                   		withDockerServer([uri: 'unix:///var/run/docker.sock']) {
                        		sh 'docker build -t messege-processor:$BUILD_NUMBER .'
					sh 'docker tag messege-processor:$BUILD_NUMBER rkudryashov/messege-processor:$BUILD_NUMBER'
					sh 'docker push rkudryashov/messege-processor:$BUILD_NUMBER'
				}
			}	
		   }
		}
	     dir (gateway){
                sh 'cp -R $JENKINS_HOME/workspace/$JOB_NAME/dir1/message-gateway/* .'
                sh 'echo \'FROM maven\\n\\n\\nCOPY . /workdir/\\nWORKDIR /workdir/\\nENTRYPOINT ["mvn"]\\nCMD ["tomcat7:run"]\' > Dockerfile'
		docker.withTool('docker'){
                        withDockerRegistry([credentialsId: 'f9b46bc9-8260-4db8-821c-b9fa96fdb4f2', url: 'https://index.docker.io/v1/']) {
                                withDockerServer([uri: 'unix:///var/run/docker.sock']) {
                                        sh 'docker build -t messege-gateway:$BUILD_NUMBER .'
                                        sh 'docker tag messege-gateway:$BUILD_NUMBER rkudryashov/messege-gateway:$BUILD_NUMBER'
                                        sh 'docker push rkudryashov/messege-gateway:$BUILD_NUMBER'

				}
          		}		
    		}	
	
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

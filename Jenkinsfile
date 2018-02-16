

import groovy.json.JsonSlurper
import groovy.json.JsonOutput
def binHost = 'http://requestbin.fullcontact.com'
def workdir = "dir1"
def art = "artifactory"
def proc = "message_processor_$BUILD_NUMBER"
def gateway = "message_gateway_$BUILD_NUMBER"
node(){
       stage('Create bin'){
       echo 'Going to create bin'
       def rbin = httpRequest(
               consoleLogResponseBody: true,
               httpMode: 'POST',
               url: "${binHost}/api/v1/bins"
               )
        def json = new JsonSlurper().parseText(rbin.getContent())
        binNumber = json.name.toString()
        println binNumber
	}
    stage('get source') {
	dir(workdir) {
		git branch: 'rkudryashov', credentialsId: '650ea460-204d-4861-963b-af80d47367b0', url: 'git@gitlab.com:nikolyabb/epam-devops-3rd-stream.git'

	 }
	}	
    stage('run tests') {
	dir(workdir) {
		withMaven(maven: 'maven'){
		   sh 'mvn clean test > maven.tests-$BUILD_NUMBER.txt'
	    }
	}
    }
    stage('build package') {
	 dir(workdir) {
                withMaven(maven: 'maven'){
                   sh 'mvn -X clean package -Dmaven.test.skip=true > /dev/null 2>$1'
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
	    dir (art){
			sh 'cp $(find $JENKINS_HOME/workspace/$JOB_NAME/dir1 -name "maven.tests-$BUILD_NUMBER.txt") .'
	     dir (proc) {
			 sh 'cp $(find $JENKINS_HOME/workspace/$JOB_NAME/dir1/message-processor/ -name "message-processor-1.0-SNAPSHOT.jar") .message-processor-1.0-SNAPSHOT.$BUILD_NUMBER.jar'
			 sh 'cp $(find $JENKINS_HOME/workspace/$JOB_NAME/dir1/message-processor/ -name "config.properties") ./config.properties.$BUILD_NUMBER'
			 sh 'echo \'FROM java:8\\n\\n\\nCOPY . /workdir/\\nWORKDIR /workdir/\\nENTRYPOINT ["java"]\\nCMD ["-jar","message-processor-1.0-SNAPSHOT.$BUILD_NUMBER.jar","config.properties.$BUILD_NUMBER"]\' > Dockerfile'
			 docker.withTool('docker') {
				 withDockerRegistry([credentialsId: 'f9b46bc9-8260-4db8-821c-b9fa96fdb4f2', url: 'https://index.docker.io/v1/']) {
					 withDockerServer([uri: 'unix:///var/run/docker.sock']) {
						 sh 'docker build -t messege-processor:$BUILD_NUMBER .'
						 sh 'docker tag messege-processor:$BUILD_NUMBER rkudryashov/messege-processor:$BUILD_NUMBER'
						 sh 'docker push rkudryashov/messege-processor:$BUILD_NUMBER'
						 sh 'docker rmi rkudryashov/messege-processor:$BUILD_NUMBER messege-processor:$BUILD_NUMBER'
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
									    sh 'docker rmi rkudryashov/messege-gateway:$BUILD_NUMBER messege-gateway:$BUILD_NUMBER'
	                                    }
						        }
				}
		 }
	  }
	}
    stage('deploy to env') {
	 docker.withTool('docker'){
                 withDockerServer([uri: 'unix:///var/run/docker.sock']) {
                   sh 'docker run -d --name message-gateway -p 8888:8080 rkudryashov/messege-gateway:$BUILD_NUMBER'
                   sh 'docker run -d --name rabbitmq --net=container:message-gateway rabbitmq'
                   sh 'docker run -d --name messege-processor --net=container:rabbitmq rkudryashov/messege-processor:$BUILD_NUMBER'
			       sleep 30
				   sh 'docker start messege-processor'
	  }
	 }
	}
    stage('provision env') {

    }
    stage('print bin info'){
        echo 'Print bin info'

    }

    stage('Send test message'){
        echo 'Going to send message'
        println "${binHost}/${binNumber}?inspect"
    }
    }

    stage('integration test') {
	echo 'Stage test'
    }
    stage('send report') {
	echo 'Stage report'
    }
}

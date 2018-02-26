#!groovy
import groovy.json.JsonSlurper


def workdir = "dir1"
def binURL = "https://requestbin.fullcontact.com"
def buildReport = "Build number ${BUILD_NUMBER} \n-----------------------------"

node(){
    stage('prepare'){
        sh "export"
        dir(workdir) {
            deleteDir()
        }
        echo 'Delete all stopped containers'
        sh 'docker ps -q -f status=exited | xargs --no-run-if-empty docker rm'
        echo ' Delete all dangling (unused) images'
        sh 'docker images -q -f dangling=true | xargs --no-run-if-empty docker rmi'
        try {
            sh 'docker ps -a'
            sh 'docker rm -f rabbitmq processor gateway'
            sh 'docker rmi -f rabbitmq message-processor:* message-gateway:*'
        }catch (error){
            echo 'Clean enviroment'
        }
    }
    stage('get source') {
        dir('sources') {
            checkout scm
        }

    }
    stage('run tests') {
        dir('sources') {
            withMaven(maven: 'maven') {
                sh 'mvn clean test'
            }

        }
    }
    stage('build package') {
        dir('sources') {
            withMaven(maven: 'maven') {
                sh 'mvn package -Dmaven.test.skip=true'
            }
        sh 'docker build -t niknestor/processor:$BUILD_NUMBER -f message-processor/Dockerfile message-processor/'
        sh 'docker build -t niknestor/gateway:$BUILD_NUMBER -f message-gateway/Dockerfile message-gateway/'
        }
    }
    stage('save artifact') {

    }
    stage('deploy to env') {
        sh 'docker run -d --name gateway --network=env -p 18080:8080 niknestor/gateway:$BUILD_NUMBER'        
	sleep 30
        sh 'docker run -d --name rabbitmq --network=env rabbitmq'
        sh 'docker run -d --name processor --network=env niknestor/processor:$BUILD_NUMBER'
        sh' docker start processor'
        sh 'docker ps'
    }
    stage('ntergration tests') {
        echo 'Going to create bin'
        def response = httpRequest(
                httpMode: 'POST',
                url: "${binURL}/api/v1/bins",
                validResponseCodes: '200',
        ).getContent()
	def binNum = new JsonSlurper().parseText(response).name.toString()
	buildReport += "Bin ${binNum} created on ${binURL}\n"
        def messages = [
            'curl http://172.18.0.4:18080/message -X POST -d \'{"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}}\'',
            'curl http://172.18.0.4:18080/message -X POST -d \'{"messageId":2, "timestamp":2234, "protocolVersion":"1.0.1", "messageData":{"mMX":1234, "mPermGen":5678, "mOldGen":22222}}\'',
            'curl http://172.18.0.4:18080/message -X POST -d \'{"messageId":3, "timestamp":3234, "protocolVersion":"2.0.0", "payload":{"mMX":1234, "mPermGen":5678, "mOldGen":22222, "mYoungGen":333333}\''
        ]
        messages.eachWithIndex{ message, i ->
                sh "docker exec gateway ${message}"
                def getLogProcessor = sh (script:"docker logs --tail 1 processor", returnStdout: true)
                i++
                assert getLogProcessor.contains("id=${i}")
                buildReport += "Test ${i}: ${getLogProcessor} \n"
        }
        httpRequest( 
            consoleLogResponseBody: true,
            httpMode: 'POST',
            url: "${binURL}/${binNum}",
            requestBody: "'${buildReport}")
        echo "Report available on ${binURL}/${binNum}?inspect"
    }
}

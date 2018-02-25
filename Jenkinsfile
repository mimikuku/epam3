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
        sh 'docker run -d --name rabbitmq rabbitmq'
        sleep 25
        sh 'docker run -d --link rabbitmq --name processor niknestor/processor:$BUILD_NUMBER'
        sh 'docker run -d --link rabbitmq --name gateway niknestor/gateway:$BUILD_NUMBER'
	sh 'docker ps'
    }
    stage('create bin') {
        echo 'Going to create bin'
        def response = httpRequest(
                httpMode: 'POST',
                url: '${binURL}/api/v1/bins',
                validResponseCodes: '200',
        ).getContent()
	def binNum = new JsonSlurper().parseText(response).name.toString()
	echo 'Bin ${binNum} created on ${binURL}'
    }
    stage('integration tests') {
        def messages = [
            'curl http://localhost:8080/message -X POST -d \'{"messageId":1, "timestamp":1234, "protocolVersion":"1.0.0", "messageData":{"mMX":1234, "mPermGen":1234}}\'',
            'curl http://localhost:8080/message -X POST -d \'{"messageId":2, "timestamp":2234, "protocolVersion":"1.0.1", "messageData":{"mMX":1234, "mPermGen":5678, "mOldGen":22222}}\'',
            'curl http://localhost:8080/message -X POST -d \'{"messageId":3, "timestamp":3234, "protocolVersion":"2.0.0", "payload":{"mMX":1234, "mPermGen":5678, "mOldGen":22222, "mYoungGen":333333}\''
        ]
        messages.eachWithIndex{ message, i ->
            try {
                sh 'docker exec gateway $message'
                def getLogProcessor = sh(script:"docker logs --tail 1 message-processor", returnStdout: true)
                assert getLogProcessor.contains('id=${i}')
                $(getlogProcessor) == '200'
            }catch (error){
                getLogProcessor = error.getMessage()
            }
            i++
            echo 'Test ${i}: ${getlogProcessor}'
            buildReport += 'Test ${i}: ${getlogProcessor}\n'
        }
    }
    stage('generate report') {
        httprequest( consoleLogResponseBody: true,
                     httpMode: 'POST',
                     url: '${binURL}/${binNum}',
                     requestBody: '$buildReport')
        echo 'Report available on ${binURL}/${binNum}?inspect'
    }
}

/* import shared library */
@Library('juliosISTY-shared-library')_

pipeline {
    environment {
        ID_DOCKER = "julios2"
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        APP_NAME = "jules"
        STAGING = "juliosISTY-staging"
        PRODUCTION = "juliosISTY-production"
        STG_API_ENDPOINT = "ip10-0-1-3-cebne3oj3e60g67j2k40-1993.direct.docker.labs.eazytraining.fr"
        STG_APP_ENDPOINT = "ip10-0-1-3-cebne3oj3e60g67j2k40-80.direct.docker.labs.eazytraining.fr"
        PROD_API_ENDPOINT = "ip10-0-1-4-cebne3oj3e60g67j2k40-1993.direct.docker.labs.eazytraining.fr"
        PROD_APP_ENDPOINT = "ip10-0-1-4-cebne3oj3e60g67j2k40-80.direct.docker.labs.eazytraining.fr"
        INTERNAL_PORT = "5000"
       EXTERNAL_PORT = "${PORT_EXPOSED}"
       CONTAINER_IMAGE = "${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }    
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t julios/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker build -t julios/$IMAGE_NAME:$IMAGE_TAG .
                        docker run -d -p 80:5000 -e PORT=5000 --name $IMAGE_NAME julios/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                    
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        curl http://localhost | grep -q "Hello world!"
                    '''
                    
                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                    
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                expression {GIT_BRANCH == 'origin/master'}
            }
            agent any

            steps {
                script {
                    sh """
                        echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
                        curl -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json 
                    """
                    
                }
            }
        }
        stage('Push image in production and deploy it') {
            when {
                expression {GIT_BRANCH == 'origin/master'}
            }
            agent any

            steps {
                script {
                    sh """
                        curl -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json' -d '{"your_name":"${APP_NAME}","container_image":"${CONTAINER_IMAGE}", "external_port":"${EXTERNAL_PORT}", "internal_port":"${INTERNAL_PORT}"}'
                    """
                    
                }
            }
        }
    }
    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        } 
    }
}

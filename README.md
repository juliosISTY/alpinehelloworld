# alpinehelloworld
Jenkins TPs: An Alpine-based Docker example 

**Used plugins**
- http_request
- github integration
- docker-buid steps
- slack notification
- role-based authaurization strategy

## Creation of free-style Jobs

### job : build 

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest


```shell
#! /bin/bash
git clone https://github.com/juliosISTY/alpinehelloworld.git
cd alpinehelloworld
docker build -t ${IMAGE_NAME}:${IMAGE_TAG} . 
```

Adding http_request plugin

### job : test acceptance

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest


```shell
#!/bin/bash
docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
sleep 5
```

HTTP Resquest
URL: http://localhost

```shell
curl http://localhost | grep -q "Hello world!"
docker stop ${IMAGE_NAME}
docker rm ${IMAGE_NAME}
```

### job : artefact

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest


```
#!/bin/bash
docker build -t julios/${IMAGE_NAME}:${IMAGE_TAG} .
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} julios/${IMAGE_NAME}:${IMAGE_TAG}
sleep 5
```

HTTP Resquest
URL: http://localhost

```shell
curl http://localhost | grep -q "Hello world!"
docker stop ${IMAGE_NAME}
docker rm ${IMAGE_NAME}
```
## Creation of pipeline Jobs

### Job: deployment (with slack notification)

```groovy
pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       STAGING = "eazytraining-staging"
       PRODUCTION = "eazytraining-production"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t eazytraining/$IMAGE_NAME:$IMAGE_TAG .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
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
      stage('Clean Container') {
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
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
        }
     }
     stage('Push image in production and deploy it') {
       when {
              expression { GIT_BRANCH == 'origin/master' }
            }
      agent any
      environment {
          HEROKU_API_KEY = credentials('heroku_api_key')
      }  
      steps {
          script {
            sh '''
              heroku container:login
              heroku create $PRODUCTION || echo "project already exist"
              heroku container:push -a $PRODUCTION web
              heroku container:release -a $PRODUCTION web
            '''
          }
        }
     }
  }
  post {
       success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
         }
      failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
          }   
    }
}
```

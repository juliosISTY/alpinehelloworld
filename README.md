# alpinehelloworld
An Alpine-based Docker example


## Creation of free-style Jobs

### job : build 

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest


(' 
#! /bin/bash
git clone https://github.com/juliosISTY/alpinehelloworld.git
cd alpinehelloworld
docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
')

Adding http_request plugin

### job : test acceptance

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest



#!/bin/bash
docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
sleep 5

HTTP Resquest
URL: http://localhost


curl http://localhost | grep -q "Hello world!"
docker stop ${IMAGE_NAME}
docker rm ${IMAGE_NAME}


### job : artefact

IMAGE_NAME: alpinehelloworld
IMAGE_TAG : latest



#!/bin/bash
docker build -t julios/${IMAGE_NAME}:${IMAGE_TAG} .
docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} julios/${IMAGE_NAME}:${IMAGE_TAG}
sleep 5

HTTP Resquest
URL: http://localhost


curl http://localhost | grep -q "Hello world!"
docker stop ${IMAGE_NAME}
docker rm ${IMAGE_NAME}

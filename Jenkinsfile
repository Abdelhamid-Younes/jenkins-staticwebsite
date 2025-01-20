/* import shared library */
@Library('shared-library')_
pipeline {
    environment {

        IMAGE_NAME = "${PARAM_IMAGE_NAME}"                    /*staticwebsite*/
        APP_NAME = "${PARAM_APP_NAME}"                        /*younesabdh*/
        IMAGE_TAG = "${PARAM_IMAGE_TAG}"                      /*latest*/
        
        STAGING = "${PARAM_APP_NAME}-staging"
        PRODUCTION = "${PARAM_APP_NAME}-prod"
        DOCKERHUB_USR = "${PARAM_DOCKERHUB_ID}"
        DOCKERHUB_PSW = credentials('dockerhub')
        APP_EXPOSED_PORT = "${PARAM_PORT_EXPOSED}"            /*80 by default*/

        STG_API_ENDPOINT = "ip10-0-5-4-cu72iu6l795g00d7mpg0-1993.direct.docker.labs.eazytraining.fr"
        STG_APP_ENDPOINT = "ip10-0-5-4-cu72iu6l795g00d7mpg0-80.direct.docker.labs.eazytraining.fr"
        PROD_API_ENDPOINT = "ip10-0-5-5-cu72iu6l795g00d7mpg0-1993.direct.docker.labs.eazytraining.fr"
        PROD_APP_ENDPOINT = "ip10-0-5-5-cu72iu6l795g00d7mpg0-80.direct.docker.labs.eazytraining.fr"
        
        INTERNAL_PORT = "${PARAM_INTERNAL_PORT}"              /*5000 by default*/
        EXTERNAL_PORT = "${PARAM_PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}"

        //EC2_PRIVATE_KEY = credentials('private_key')
        SSH_USER = "ubuntu"
        STAGING_IP = "35.174.106.205"
        PROD_IP = "35.174.106.205"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t ${DOCKERHUB_USR}/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on built image'){
            agent any
            steps {
                script{
                    sh '''
                        echo "Cleaning existing container if exists"
                        docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                        docker run --network jenkins_jenkins_network --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$INTERNAL_PORT ${DOCKERHUB_USR}/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps{
                script {
                    sh 'sleep 5'
                    sh 'curl -k $IMAGE_NAME:$APP_EXPOSED_PORT | grep -i "Dimension"'
                    sh 'if [ $? -eq 0 ]; then echo "Acceptance test succeeded"; fi'  // // Verify the test
                }
            }
        }
        stage('Clean container') {
            agent any
            steps{
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                }
            }
        }
        stage('Login and Push Image on Docker Hub') {
            when{
                expression {GIT_BRANCH == 'origin/master'}
            }
            agent any
            steps{
                script {
                    sh '''
                        echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                        docker push $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
        // stage('STAGING - Deploy app') {
        //     agent any
        //     steps {
        //         script {
        //             sh """
        //                 echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
        //                 curl -k -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
        //             """
        //         }
        //     }
        // }

        // stage('PROD - Deploy app') {
        //     when {
        //         expression { GIT_BRANCH == 'origin/master' }
        //     }
        //     agent any
        //     steps {
        //         script {
        //         sh """
        //             echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
        //             curl -k -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
        //         """
        //         }
        //     }
        // }

        stage('STAGING - Deploy on EC2') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Connecting to the staging EC2 instance and deploying the container"
                        ssh -o StrictHostKeyChecking=no -i $EC2_PRIVATE_KEY $SSH_USER@<$STAGING_IP> << EOF
                            docker pull $CONTAINER_IMAGE
                            docker stop $IMAGE_NAME || true
                            docker rm $IMAGE_NAME || true
                            docker run --name $IMAGE_NAME -d -p $EXTERNAL_PORT:$INTERNAL_PORT $CONTAINER_IMAGE
                        EOF
                    '''
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
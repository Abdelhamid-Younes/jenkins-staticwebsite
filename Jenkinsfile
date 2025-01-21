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
        
        INTERNAL_PORT = "${PARAM_INTERNAL_PORT}"              /*5000 by default*/
        EXTERNAL_PORT = "${PARAM_PORT_EXPOSED}"
        CONTAINER_IMAGE = "${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}"

        SSH_USER = "ubuntu"
        STAGING_IP = "3.88.8.175"
        PROD_IP = "98.82.5.111"
    }
    agent any
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

        stage('Deploy STAGING on AWS EC2') {
            steps {
                script {
                    sshagent(credentials: ['private_key']) {
                        sh '''
                            echo "Connecting to the staging EC2 instance"
                            echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                            
                            ssh -o StrictHostKeyChecking=no $SSH_USER@$STAGING_IP "
                                docker pull $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                                docker stop $IMAGE_NAME || true
                                docker rm $IMAGE_NAME || true
                                sleep 15
                                docker run --rm --name $IMAGE_NAME -d -p $EXTERNAL_PORT:$INTERNAL_PORT ${DOCKERHUB_USR}/$IMAGE_NAME:$IMAGE_TAG
                            "
                            
                            curl http://$STAGING_IP:$EXTERNAL_PORT | grep -i "Dimension"
                        '''
                    }
                }

            }
        }

        stage('Deploy PROD on AWS EC2') {
            steps {
                script {
                    // Prompt for confirmation before deploying on PROD environment.
                    input message: "Do you confirm Deploying on AWS PROD environment ?", ok: 'Yes'
                    
                    sshagent(credentials: ['private_key']) {
                        sh '''
                            echo "Connecting to the staging EC2 instance"
                            echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                            
                            ssh -o StrictHostKeyChecking=no $SSH_USER@$PROD_IP "
                                docker pull $DOCKERHUB_USR/$IMAGE_NAME:$IMAGE_TAG
                                docker stop $IMAGE_NAME || true
                                docker rm $IMAGE_NAME || true
                                sleep 15
                                docker run --rm --name $IMAGE_NAME -d -p $EXTERNAL_PORT:$INTERNAL_PORT ${DOCKERHUB_USR}/$IMAGE_NAME:$IMAGE_TAG
                            "
                            
                            curl http://$PROD_IP:$EXTERNAL_PORT | grep -i "Dimension"
                        '''
                    }
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
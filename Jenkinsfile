// import library
@Library('slack_Notifier')_
pipeline{

    environment{
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "${BUILD_TAG}"
        USERNAME = "sadofrazer"
        CONTAINER_NAME = "alpinehelloworld"
        STAGING = "frazer-staging-env"
        PRODUCTION ="frazer-prod-env"
        EC2_PRODUCTION_HOST= "52.70.122.139"
    }

    agent any

    stages{

        stage ('Build image'){
            agent{ label 'test'}
            steps{
                script{
                    sh 'docker build -t ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage ('Run a container and Test Image'){
            agent{ label 'test'}
            steps{
                script{
                    sh '''
                       docker stop ${CONTAINER_NAME} || true
                       docker rm ${CONTAINER_NAME} || true
                       docker run -d --name ${CONTAINER_NAME} -e PORT=5000 -p 5000:5000 ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                       sleep 5
                       curl http://localhost:5000 | grep -iq "Hello world! AJC"
                    '''
                }
            }
        }

        stage ('save artifact and clean env'){
            agent{ label 'test'}
            environment{
                PASSWORD = credentials('dockerhub_password')
            }
            steps{
                script{
                    sh '''
                       docker login -u ${USERNAME} -p ${PASSWORD}
                       docker push ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                       docker stop ${CONTAINER_NAME}
                       docker rm ${CONTAINER_NAME}
                       docker rmi ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage ('deploy app on staging env'){
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh '''
                       heroku container:login
                       heroku create $STAGING || echo "project already exist"
                       heroku container:push -a $STAGING web
                       heroku container:release -a $STAGING web
                       
                    '''
                    slackSend (color: '#00F000', message: "TEST: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                }
            }
        }

        stage ('deploy app on preprod env'){
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh '''
                       heroku container:login
                       heroku create $PRODUCTION || echo "project already exist"
                       heroku container:push -a $PRODUCTION web
                       heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }

        stage ('deploy app on Prod env'){
            agent any
            when {
                expression { GIT_BRANCH == 'origin/master'}
            }
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_prod_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    script{
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                        }	
                        sh '''
                           ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG 
                        '''
                    }
                }
            }
        }

    }

    post {
        always{
            script{
                slackNotifier currentBuild.result
            }
        }
    }

}
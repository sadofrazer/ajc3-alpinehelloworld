pipeline{

    environment{
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        USERNAME = "sadofrazer"
        CONTAINER_NAME = "alpinehelloworld"
        STAGING = "frazer-staging-env"
        PRODUCTION ="frazer-prod-env"
    }

    agent any

    stages{

        stage ('Build image'){
            steps{
                script{
                    sh 'docker build ${USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage ('Run a container and Test Image'){
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
            agent any
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


    }
}
pipeline{


// Définition des variables d'environnement
environment{
    IMAGE_NAME = 'webstatic'
    PORT_EXPOSED = '${PORT_EXPOSED}'
    URL_REGISTRY = 'https://registry.iforce5demo.com/'
    IMAGE_TAG = "1.0"
    SSH_USER_RELEASE = "root"
    SSH_USER_STAGING = "administrator"
    SSH_HOST_STAGING = "192.168.100.32"
    SSH_HOST_RELEASE = "192.168.100.32"
    DOCKERHUB_AUTH = credentials('DOCKER_HUB')
}

// Aucun agent spécifique, cela signifie que les étapes peuvent être exécutées sur n'importe quel agent disponible
agent none

options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'));
   }

parameters {
       string(name: 'Instance', defaultValue: 'staging', description: 'Configuration de la construction (staging ou release)' )
   }

// Définition des étapes du pipeline
stages {

    // Etape de construction de l'image Docker
    stage ('Build image') {
        agent any
        steps {
            script {
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }
    }

    // Etape de démarrage du conteneur basé sur l'image construite
    stage ('Run container based on build images') {
        agent any
        steps {
            script {
                sh '''
                    echo "Clean Environment"
                    docker rm -f ${IMAGE_NAME} || echo "container does not exist"
                    docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:80 -e PORT=80 ${URL_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 5
                '''
            }
        }
    }

    // Etape de test de l'image en vérifiant si elle renvoie 'hello world!' lorsqu'on accède à son URL
    stage ('Test image') {
        agent any
        steps {
            script {
                sh '''
                    curl http://172.17.0.1:${PORT_EXPOSED} | grep -i 'Welcome!'
                '''
            }
        }
    }

    // Etape de nettoyage du conteneur
    stage ('Clean Container') {
        agent any
        steps {
            script {
                sh '''
                    docker rm -f ${IMAGE_NAME}
                '''
            }
        }
    }

    // Etape de connexion au registre Docker et de pousser l'image construite
    stage ('Login docker container et push') {
        agent any
        steps {
            script {
                sh '''
                   echo ${DOCKERHUB_AUTH_PSW} | docker login ${REGISTRY_URL} -u ${DOCKERHUB_AUTH_USR} --password-stdin
                   docker push ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}
                  '''
            }
        }
    }

    stage('Deploy in staging') {
        when {
            expression { env.GIT_BRANCH == 'origin/main' && params.Instance == 'staging' }
        }
        steps {
            sshagent(credentials: ['SSH_FORGE']) {
                sh '''
                    [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                    ssh-keyscan -t rsa,dsa ${SSH_HOST_STAGING} >> ~/.ssh/known_hosts
                    command1="echo ${DOCKERHUB_AUTH_PSW} | docker login ${REGISTRY_URL} -u ${DOCKERHUB_AUTH_USR} --password-stdin"
                    command2="docker stop ${IMAGE_TAG} || echo 'container does not exist'"
                    command3="docker rm ${IMAGE_TAG} || echo 'container does not exist'"
                    command4="docker rmi ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} || echo 'container does not exist'"
                    command5="docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:80 -e PORT=80 ${URL_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    command6="sleep 5"
                    ssh -o SendEnv=${IMAGE_NAME} \
                        -o SendEnv=${IMAGE_TAG} \
                        -o SendEnv=${REGISTRY_URL} \
                        -o SendEnv=${DOCKERHUB_AUTH_PSW} \
                        -o SendEnv=${DOCKERHUB_AUTH_USR} \
                        ${SSH_USER_STAGING}@${SSH_HOST_STAGING} \
                        "$command1 && $command2 && $command3 && $command4 && $command5 && $command6"
                '''
            }
        }
    }

    stage('Deploy in release') {
         when {
              expression { env.GIT_BRANCH == 'origin/main' && params.Instance == 'release' }
         }
         steps {
             sshagent(credentials: ['SSH_FORGE']) {
                  sh '''
                      [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
                      ssh-keyscan -t rsa,dsa ${SSH_HOST_RELEASE} >> ~/.ssh/known_hosts
                      command1="echo ${DOCKERHUB_AUTH_PSW} | docker login ${REGISTRY_URL} -u ${DOCKERHUB_AUTH_USR} --password-stdin"
                      command2="docker stop ${IMAGE_TAG} || echo 'container does not exist'"
                      command3="docker rm ${IMAGE_TAG} || echo 'container does not exist'"
                      command4="docker rmi ${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG} || echo 'container does not exist'"
                      command5="docker run --name ${IMAGE_NAME} -d -p ${PORT_EXPOSED}:80 -e PORT=80 ${URL_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                      command6="sleep 5"
                      ssh -o SendEnv=${IMAGE_NAME} \
                          -o SendEnv=${IMAGE_TAG} \
                          -o SendEnv=${REGISTRY_URL} \
                          -o SendEnv=${DOCKERHUB_AUTH_PSW} \
                          -o SendEnv=${DOCKERHUB_AUTH_USR} \
                          ${SSH_USER_RELEASE}@${SSH_HOST_RELEASE} \
                          "$command1 && $command2 && $command3 && $command4 && $command5 && $command6"
                  '''
             }
         }
    }

    stage('Test Prod') {
        when {
            expression { env.GIT_BRANCH == 'origin/main' && params.Instance == 'release' }
        }
        steps {
            sh '''
                sleep 3
                curl http://${SSH_HOST_RELEASE}:${PORT_EXPOSED}
            '''
        }
    }

    stage('Test Staging') {
        when {
            expression { env.GIT_BRANCH == 'origin/main' && params.Instance == 'staging' }
        }
        steps {
            sh '''
                sleep 3
                curl http://${SSH_HOST_STAGING}:${PORT_EXPOSED}
            '''
        }
    }

}

// Actions à effectuer après l'exécution du pipeline
post {
    success {
        // Envoie d'un message de succès à Slack avec des liens vers l'application en production et en staging
        slackSend (color: '#00FF00', message: "NAME - SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - PROD URL => http://${PROD_APP_ENDPOINT} , STAGING URL => http://${STG_APP_ENDPOINT}")
    }
    failure {
        // Envoie d'un message d'échec à Slack en cas d'échec du pipeline
        slackSend (color: '#FF0000', message: "NAME - FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
}


}


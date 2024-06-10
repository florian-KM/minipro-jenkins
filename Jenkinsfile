// Définition des variables d'environnement
environment{
    IMAGE_NAME = 'webstatic'
    IMAGE_TAG = 'latest'
    PORT_EXPOSED = '${PORT_EXPOSED}'
    URL_REGISTRY = 'https://registry.iforce5demo.com/'
    STAGING = ''
    PRODUCTION = ''
}

// Aucun agent spécifique, cela signifie que les étapes peuvent être exécutées sur n'importe quel agent disponible
agent none

// Définition des étapes du pipeline
stages {

    // Etape de construction de l'image Docker
    stage ('Build image') {
        agent any
        steps {
            script {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
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
                    docker rm -f $IMAGE_NAME || echo "container does not exist"
                    docker run --name $IMAGE_NAME -d -p $PORT_EXPOSED:5000 -e PORT=5000 eazytraining/$IMAGE_NAME:$IMAGE_TAG
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
                    curl http://172.17.0.1:$PORT_EXPOSED | grep -q 'hello world!'
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
                    docker stop $IMAGE_NAME
                    docker rm $IMAGE_NAME
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
                    echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
                    docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
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

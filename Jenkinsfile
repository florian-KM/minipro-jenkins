@Library('shared-librairy')_
pipeline{
    // Définition des variables d'environnement
    environment{
        IMAGE_NAME = 'webstatic'
        IMAGE_TAG = 'latest'
        // PORT_EXPOSED = '${PORT_EXPOSED}'
        URL_REGISTRY = 'registry.iforce5demo.com'
        STAGING = ''
        PRODUCTION = ''
    }

    // Aucun agent spécifique, cela signifie que les étapes peuvent être exécutées sur n'importe quel agent disponible
    agent none

    // Configuration des options du pipeline
    options{
        // Utilisation d'un logRotator pour gérer la suppression des anciens builds et artefacts
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
    }

    // Définition des outils utilisés par le pipeline
    // tools {
    //     // Spécification de l'outil Git à utiliser, avec le nom 'git-jenkins'
    //     git 'git-jenkins'
    // }


    // Définition des étapes du pipeline
    stages {

        // Etape de construction de l'image Docker
        stage ('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t $URL_REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        // Etape de démarrage du conteneur basé sur l'image construite
        stage ('Run container based on build images') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Nettoyer l'environnement"
                        docker rm -f $IMAGE_NAME || echo "Mon container n'existe pas"
                        docker run --name $IMAGE_NAME -d -p $PORT_EXPOSED:5000 -e PORT=5000 $URL_REGISTRY/$IMAGE_NAME:$IMAGE_TAG
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
                        curl http://172.17.0.1:$PORT_EXPOSED 
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
        always {
            script {
                notification-slack currentBuild.result
            }
        } 
    }

}

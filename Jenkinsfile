
//     //Deployer
// stage ('Deploy in staging') {
//     when {
//         expression { GIT_BRANCH == 'main' }
//     }
//     steps {
//             sshagent(credentials: [SSH_FORGE]) {
//                 sh '''
//                     apk add --no-cache openssh-client
//                     eval $(ssh-agent -s)
//                     mkdir -p ~/.ssh
//                     echo "${SSH_KEY}" | tr -d '\r' > ~/.ssh/id_rsa
//                     chmod 600 ~/.ssh/id_rsa
//                     ssh-keygen -lf ~/.ssh/id_rsa # Vérifie la clé pour s'assurer qu'elle est bien formée
//                     ssh-keyscan -H ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
//                     echo " bonne nouvelle "
//                 '''
//             }
//     }
// }



pipeline {
    agent any

    environment {
        SSH_CREDENTIALS_ID = 'SSH_FORGE' // Remplacez par l'ID de vos credentials SSH
        REMOTE_HOST = '89.117.51.226' // Remplacez par l'adresse de votre hôte distant
    }

    stages {
        stage('SSH Connection') {
            steps {
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh 'ssh -o StrictHostKeyChecking=no root@${REMOTE_HOST} "echo Hello, World!"'
                }
            }
        }
    }
}

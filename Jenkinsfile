pipeline {
    agent any
    
    environment {
        // Configuration Docker
        DOCKER_IMAGE = "aminamahrour/hello-world:${BUILD_NUMBER}"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"
        
        // Configuration Ansible
        ANSIBLE_USER = "aminamahrour"
        ANSIBLE_HOST = "192.168.1.100"  // À remplacer par votre IP Ansible
        ANSIBLE_SSH_CREDENTIALS = "ansible-ssh"  // Doit correspondre à l'ID dans Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // Récupère automatiquement le code du dépôt GitHub
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construit l'image avec le Dockerfile du dépôt
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh """
                        # Connexion sécurisée à Docker Hub
                        docker login -u "${DOCKER_USER}" -p "${DOCKER_PASS}" ${DOCKER_REGISTRY}
                        
                        # Push de l'image
                        docker push "${DOCKER_IMAGE}"
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent([ANSIBLE_SSH_CREDENTIALS]) {
                    sh """
                        # Connexion SSH sécurisée sans vérification de host key
                        ssh -o StrictHostKeyChecking=no \
                            -o UserKnownHostsFile=/dev/null \
                            ${ANSIBLE_USER}@${ANSIBLE_HOST} \
                            'ansible-playbook /home/ubuntu/deploy.yml -e image_version=${DOCKER_IMAGE}'
                    """
                }
            }
        }
    }

    post {
        always {
            // Nettoyage sécurisé
            sh 'docker logout ${DOCKER_REGISTRY}'
            cleanWs()  // Nettoie l'espace de travail Jenkins
        }
        success {
            // Notification en cas de succès (requiert plugin Slack)
            slackSend(
                color: 'good', 
                message: "SUCCESS: Pipeline ${BUILD_NUMBER} - ${env.JOB_NAME}"
            )
        }
        failure {
            // Notification en cas d'échec
            slackSend(
                color: 'danger',
                message: "FAILED: Pipeline ${BUILD_NUMBER} - ${env.JOB_NAME}"
            )
        }
    }
}

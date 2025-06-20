pipeline {
    agent any

    environment {
        // Image Docker tag
        DOCKER_IMAGE = "aminamahrour/hello-world:${BUILD_NUMBER}"
        DOCKER_REGISTRY = "https://index.docker.io/v1/"

        // Ansible (VM)
        ANSIBLE_USER = "ubuntu"
        ANSIBLE_HOST = "10.132.0.4"  // IP interne ou publique correcte
        ANSIBLE_SSH_CREDENTIALS = "ansible-ssh"  // ID exact défini dans Jenkins
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
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
                        docker login -u "${DOCKER_USER}" -p "${DOCKER_PASS}" ${DOCKER_REGISTRY}
                        docker push "${DOCKER_IMAGE}"
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sshagent([env.ANSIBLE_SSH_CREDENTIALS]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
                        ${ANSIBLE_USER}@${ANSIBLE_HOST} \
                        'ansible-playbook /home/ubuntu/deploy.yml -e image_version=${DOCKER_IMAGE}'
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout ${DOCKER_REGISTRY}'
            cleanWs()
        }

        success {
            slackSend(
                color: 'good',
                message: "✅ SUCCESS: Pipeline #${BUILD_NUMBER} - ${env.JOB_NAME}"
            )
        }

        failure {
            slackSend(
                color: 'danger',
                message: "❌ FAILURE: Pipeline #${BUILD_NUMBER} - ${env.JOB_NAME}"
            )
        }
    }
}

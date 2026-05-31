pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "shashank1411"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Dharma-Raghava-Sai-Shashank/devops-project.git'
            }
        }

        stage('Set Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()

                    env.API_IMAGE    = "${env.DOCKERHUB_USER}/multi-api:${env.IMAGE_TAG}"
                    env.CLIENT_IMAGE = "${env.DOCKERHUB_USER}/multi-client:${env.IMAGE_TAG}"
                    env.WORKER_IMAGE = "${env.DOCKERHUB_USER}/multi-worker:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Build Images') {
            steps {
                sh """
                docker build -t ${API_IMAGE} ./server
                docker build -t ${CLIENT_IMAGE} ./client
                docker build -t ${WORKER_IMAGE} ./worker
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    '''
                }
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push ${API_IMAGE}
                docker push ${CLIENT_IMAGE}
                docker push ${WORKER_IMAGE}
                """
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {

                    sh """
                    rm -rf devops-charts
                    git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/Dharma-Raghava-Sai-Shashank/devops-charts.git

                    cd devops-charts

                    # ---------------- CLIENT ----------------
                    sed -i "s|image-tag:.*|image-tag: \"${IMAGE_TAG}\"|g" devops-project/client-deployment.yaml

                    # ---------------- API ----------------
                    sed -i "s|image-tag:.*|image-tag: \"${IMAGE_TAG}\"|g" devops-project/server-deployment.yaml

                    # ---------------- WORKER ----------------
                    sed -i "s|image-tag:.*|image-tag: \"${IMAGE_TAG}\"|g" devops-project/worker-deployment.yaml

                    git config user.email "jenkins@local"
                    git config user.name "jenkins"

                    git add .
                    git commit -m "Update image tags to ${IMAGE_TAG}" || echo "No changes"
                    git push
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline SUCCESS 🚀"
        }

        failure {
            echo "Pipeline FAILED ❌"
        }
    }
}
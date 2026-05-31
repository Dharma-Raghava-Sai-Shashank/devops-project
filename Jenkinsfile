pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "shashank1411"

        // SAFE fallback if GIT_COMMIT is null
        IMAGE_TAG = "${env.GIT_COMMIT != null ? env.GIT_COMMIT.take(7) : 'latest'}"

        API_IMAGE    = "${DOCKERHUB_USER}/multi-api:${IMAGE_TAG}"
        CLIENT_IMAGE = "${DOCKERHUB_USER}/multi-client:${IMAGE_TAG}"
        WORKER_IMAGE = "${DOCKERHUB_USER}/multi-worker:${IMAGE_TAG}"
        NGINX_IMAGE  = "${DOCKERHUB_USER}/multi-nginx:${IMAGE_TAG}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Tests') {
            steps {
                sh 'echo "Run tests here (api/client/worker)"'
            }
        }

        stage('Build API Image') {
            steps {
                sh "docker build -t $API_IMAGE ./server"
            }
        }

        stage('Build Client Image') {
            steps {
                sh "docker build -t $CLIENT_IMAGE ./client"
            }
        }

        stage('Build Worker Image') {
            steps {
                sh "docker build -t $WORKER_IMAGE ./worker"
            }
        }

        stage('Build NGINX Image') {
            steps {
                sh "docker build -t $NGINX_IMAGE ./nginx"
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
                docker push $API_IMAGE
                docker push $CLIENT_IMAGE
                docker push $WORKER_IMAGE
                docker push $NGINX_IMAGE
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

                    sed -i.bak 's|multi-api:.*|multi-api:${IMAGE_TAG}|g' devops-project/server-deployment.yaml
                    sed -i.bak 's|multi-client:.*|multi-client:${IMAGE_TAG}|g' devops-project/client-deployment.yaml
                    sed -i.bak 's|multi-worker:.*|multi-worker:${IMAGE_TAG}|g' devops-project/worker-deployment.yaml
                    sed -i.bak 's|multi-nginx:.*|multi-nginx:${IMAGE_TAG}|g' devops-project/nginx-deployment.yaml

                    git config user.email "jenkins@local"
                    git config user.name "jenkins"

                    git add .
                    git commit -m "Update images ${IMAGE_TAG}" || echo "No changes to commit"
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
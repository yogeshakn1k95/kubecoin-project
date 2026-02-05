pipeline {
    agent any

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        DOCKERHUB_USER  = 'yogeshakn1k95'
        BACKEND_IMAGE   = 'kubecoin-backend'
        FRONTEND_IMAGE  = 'kubecoin-frontend'
        KUBECONFIG      = credentials('kubeconfig')
    }

    stages {

        stage('Set Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'DEV') {
                        env.ENV = 'dev1'
                        env.NAMESPACE = 'dev1'
                    } else if (env.BRANCH_NAME == 'TESTING') {
                        env.ENV = 'testing'
                        env.NAMESPACE = 'testing'
                    } else if (env.BRANCH_NAME == 'PRODUCTION') {
                        env.ENV = 'prod'
                        env.NAMESPACE = 'production'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    env.TAG = "${ENV}-${BUILD_NUMBER}"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$BACKEND_IMAGE:$TAG backend/
                docker build -t $DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG frontend/
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh """
                echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
                docker push $DOCKERHUB_USER/$BACKEND_IMAGE:$TAG
                docker push $DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                sed -i 's|IMAGE_BACKEND|$DOCKERHUB_USER/$BACKEND_IMAGE:$TAG|g' k8s/level-1-basic/backend-deployment.yaml
                sed -i 's|IMAGE_FRONTEND|$DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG|g' k8s/level-1-basic/frontend-deployment.yaml

                kubectl apply -n $NAMESPACE -f k8s/level-1-basic/
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful to namespace: ${NAMESPACE}"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}

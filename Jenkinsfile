pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yogeshakn1k95/myapp"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'echo "Run your tests here"'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ./backend"
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds_kube',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE:$DOCKER_TAG
                    """
                }
            }
        }

            stage('Update Deployment File') {
                steps {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )]) {
                        sh '''
                        git checkout main
                        git pull origin main
            
                        git config user.name "jenkins"
                        git config user.email "jenkins@test.com"
            
                        git add deployment.yaml
            
                        if git diff --cached --quiet; then
                          echo "No changes detected"
                        else
                          git commit -m "Updated image to ${DOCKER_TAG}"
                          git push https://$GIT_USER:$GIT_PASS@github.com/yogeshakn1k95/kubecoin-project.git HEAD:main
                        fi
                        '''
                    }
                }
            }
        // stage('Update Deployment File') {
        //     steps {
        //         // sh """
        //         // sed -i 's|image: .*|image: $DOCKER_IMAGE:$DOCKER_TAG|' deployment.yaml
        //         // """
        //         // sh """
        //         // git config user.name "jenkins"
        //         // git config user.email "jenkins@test.com"
        //         // git add deployment.yaml
        //         // git commit -m "Updated image to $DOCKER_TAG"
        //         // """
        //         sh '''
        //         git checkout main
        //         git pull origin main
                
        //         git config user.name "jenkins"
        //         git config user.email "jenkins@test.com"
                
        //         git add deployment.yaml
        //         git commit -m "Updated image to ${DOCKER_TAG}" || echo "No changes to commit"
        //         git push origin main || echo "Nothing to push"
        //         '''
        //         sh "git push origin main"
        //     }
        // }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
            }
        }
    }
}

// pipeline {
//     agent any

//     environment {
//         DOCKERHUB_CREDS = credentials('dockerhub-creds')
//         DOCKERHUB_USER  = 'yogeshakn1k95'
//         BACKEND_IMAGE   = 'kubecoin-backend'
//         FRONTEND_IMAGE  = 'kubecoin-frontend'
//         KUBECONFIG      = credentials('kubeconfig')
//     }

//     stages {

//         stage('Set Environment') {
//             steps {
//                 script {
//                     if (env.BRANCH_NAME == 'DEV') {
//                         env.ENV = 'dev'
//                         env.NAMESPACE = 'dev'
//                     } else if (env.BRANCH_NAME == 'TESTING') {
//                         env.ENV = 'testing'
//                         env.NAMESPACE = 'testing'
//                     } else if (env.BRANCH_NAME == 'PRODUCTION') {
//                         env.ENV = 'prod'
//                         env.NAMESPACE = 'production'
//                     } else {
//                         error "Unsupported branch: ${env.BRANCH_NAME}"
//                     }

//                     env.TAG = "${ENV}-${BUILD_NUMBER}"
//                 }
//             }
//         }

//         stage('Build Docker Images') {
//             steps {
//                 sh """
//                 docker build -t $DOCKERHUB_USER/$BACKEND_IMAGE:$TAG backend/
//                 docker build -t $DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG frontend/
//                 """
//             }
//         }

//         stage('Push to DockerHub') {
//             steps {
//                 sh """
//                 echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin
//                 docker push $DOCKERHUB_USER/$BACKEND_IMAGE:$TAG
//                 docker push $DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG
//                 """
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 sh """
//                 sed -i 's|IMAGE_BACKEND|$DOCKERHUB_USER/$BACKEND_IMAGE:$TAG|g' k8s/level-1-basic/backend-deployment.yaml
//                 sed -i 's|IMAGE_FRONTEND|$DOCKERHUB_USER/$FRONTEND_IMAGE:$TAG|g' k8s/level-1-basic/frontend-deployment.yaml

//                 kubectl apply -n $NAMESPACE -f k8s/level-1-basic/
//                 """
//             }
//         }
//     }

//     post {
//         success {
//             echo "✅ Deployment successful to namespace: ${NAMESPACE}"
//         }
//         failure {
//             echo "❌ Deployment failed"
//         }
//     }
// }

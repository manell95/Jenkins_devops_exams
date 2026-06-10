pipeline {
    agent any
    environment {
        DOCKER_ID = "nickmanell"
        DOCKER_IMAGE_MOVIE = "movie-service"
        DOCKER_IMAGE_CAST = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
        KUBECONFIG = credentials("config")
    }
    stages {
        stage('Build Movie Service') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service
                    '''
                }
            }
        }
        stage('Build Cast Service') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG ./cast-service
                    '''
                }
            }
        }
        stage('Push Movie Service') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Push Cast Service') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy Dev') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app ./charts/. \
                        --values=./charts/values.yaml \
                        --namespace dev \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy QA') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app ./charts/. \
                        --values=./charts/values.yaml \
                        --namespace qa \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy Staging') {
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app ./charts/. \
                        --values=./charts/values.yaml \
                        --namespace staging \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploy Prod') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Déployer en production ?', ok: 'Oui'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    helm upgrade --install app ./charts/. \
                        --values=./charts/values.yaml \
                        --namespace prod \
                        --set movieService.image.tag=$DOCKER_TAG \
                        --set castService.image.tag=$DOCKER_TAG
                    '''
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}

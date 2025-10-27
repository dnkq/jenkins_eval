pipeline {    
    environment {
        DOCKER_ID = "dnkq"
        DOCKER_IMAGE_CAST = "cast-service"
        DOCKER_IMAGE_MOVIE = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG cast-service/
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG movie-service/
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker run') {
            steps {
                script {
                    sh '''
                    docker rm -f cast || true
                    docker rm -f movie || true
                    docker run -d -p 80:8000 --name cast $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker run -d -p 81:8000 --name movie $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') { 
            steps {
                script {
                    sh '''
                    docker compose down || true
                    docker compose up -d
                    sleep 6
                    curl localhost:80/api/v1/checkapi
                    curl localhost:81/api/v1/checkapi
                    docker compose down
                    '''
                }
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm upgrade --install cast-service charts --values=values.yml --namespace dev
                    helm test cast-service --namespace dev

                    helm upgrade --install movie-service charts --values=values.yml --namespace dev
                    helm test movie-service --namespace dev
                    '''
                }
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm upgrade --install cast-service charts --values=values.yml --namespace qa
                    helm test cast-service --namespace qa

                    helm upgrade --install movie-service charts --values=values.yml --namespace qa
                    helm test movie-service --namespace qa
                    '''
                }
            }
        }

        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm upgrade --install cast-service charts --values=values.yml --namespace staging
                    helm test cast-service --namespace staging

                    helm upgrade --install movie-service charts --values=values.yml --namespace staging
                    helm test movie-service --namespace staging
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config

                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

                    helm upgrade --install cast-service charts --values=values.yml --namespace prod
                    helm test cast-service --namespace prod

                    helm upgrade --install movie-service charts --values=values.yml --namespace prod
                    helm test movie-service --namespace prod
                    '''
                }
            }
        }
    }
}
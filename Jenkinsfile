pipeline {
    agent any

    environment {
        DOCKER_ID  = "moussabazig"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Build') {
            steps {
                sh 'docker build -t $DOCKER_ID/movie-service:$DOCKER_TAG ./movie-service'
                sh 'docker build -t $DOCKER_ID/cast-service:$DOCKER_TAG ./cast-service'
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        docker push ${DOCKER_ID}/movie-service:${DOCKER_TAG}
                        docker push ${DOCKER_ID}/cast-service:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy dev') {
            steps {
                sh '''
                helm upgrade --install movie ./charts \
                    --namespace dev \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG
                '''
            }
        }

        stage('Deploy qa') {
            steps {
                sh '''
                helm upgrade --install movie ./charts \
                    --namespace qa \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG
                '''
            }
        }

        stage('Deploy staging') {
            steps {
                sh '''
                helm upgrade --install movie ./charts \
                    --namespace staging \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG
                '''
            }
        }

        stage('Deploy prod') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Déployer en production ?'
                sh '''
                helm upgrade --install movie ./charts \
                    --namespace prod \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG
                '''
            }
        }

    }
}

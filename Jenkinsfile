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
                helm upgrade --install movie-db bitnami/postgresql \
                    --namespace dev \
                    --set auth.username=movie_db_username \
                    --set auth.password=movie_db_password \
                    --set auth.database=movie_db_dev

                helm upgrade --install movie ./charts \
                    --namespace dev \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG \
                    --set service.nodePort=30007 \
                    --set env[0].name=DATABASE_URI \
                    --set "env[0].value=postgresql://movie_db_username:movie_db_password@movie-db-postgresql/movie_db_dev" \
                    --set env[1].name=CAST_SERVICE_HOST_URL \
                    --set "env[1].value=http://cast-service:8000/api/v1/casts/"
                '''
            }
        }

        stage('Deploy qa') {
            steps {
                sh '''
                helm upgrade --install movie-db bitnami/postgresql \
                    --namespace qa \
                    --set auth.username=movie_db_username \
                    --set auth.password=movie_db_password \
                    --set auth.database=movie_db_dev

                helm upgrade --install movie ./charts \
                    --namespace qa \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG \
                    --set service.nodePort=30008 \
                    --set env[0].name=DATABASE_URI \
                    --set "env[0].value=postgresql://movie_db_username:movie_db_password@movie-db-postgresql/movie_db_dev" \
                    --set env[1].name=CAST_SERVICE_HOST_URL \
                    --set "env[1].value=http://cast-service:8000/api/v1/casts/"
                '''
            }
        }

        stage('Deploy staging') {
            steps {
                sh '''
                helm upgrade --install movie-db bitnami/postgresql \
                    --namespace staging \
                    --set auth.username=movie_db_username \
                    --set auth.password=movie_db_password \
                    --set auth.database=movie_db_dev

                helm upgrade --install movie ./charts \
                    --namespace staging \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG \
                    --set service.nodePort=30009 \
                    --set env[0].name=DATABASE_URI \
                    --set "env[0].value=postgresql://movie_db_username:movie_db_password@movie-db-postgresql/movie_db_dev" \
                    --set env[1].name=CAST_SERVICE_HOST_URL \
                    --set "env[1].value=http://cast-service:8000/api/v1/casts/"
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
                helm upgrade --install movie-db bitnami/postgresql \
                    --namespace prod \
                    --set auth.username=movie_db_username \
                    --set auth.password=movie_db_password \
                    --set auth.database=movie_db_dev

                helm upgrade --install movie ./charts \
                    --namespace prod \
                    --set image.repository=$DOCKER_ID/movie-service \
                    --set image.tag=$DOCKER_TAG \
                    --set service.nodePort=30010 \
                    --set env[0].name=DATABASE_URI \
                    --set "env[0].value=postgresql://movie_db_username:movie_db_password@movie-db-postgresql/movie_db_dev" \
                    --set env[1].name=CAST_SERVICE_HOST_URL \
                    --set "env[1].value=http://cast-service:8000/api/v1/casts/"
                '''
            }
        }
    }
}

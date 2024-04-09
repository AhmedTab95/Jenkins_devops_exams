pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "ahmedtab95" // replace this with your docker-id
DOCKER_CAST = "cast"
DOCKER_MOVIE = "movie"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to incremen>
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 cd cast-service
                 docker build -t $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG .
                sleep 6
                '''
                }
                script {
                sh '''
                 cd movie-service
                 docker build -t $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG .
                sleep 6
                '''
                }
                
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d -p 80:8000 --name cast-service $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                    script {
                    sh '''
                    docker run -d -p 80:8000 --name movie-service $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG
                    sleep 10
                    '''
                    }
                }
            }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl http://localhost:8080/api/v1/casts/docs 
                    '''
                    }
                    script {
                    sh '''
                    curl http://localhost:8080/api/v1/movies/docs 
                    '''
                    }
            }
        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_CAST:$DOCKER_TAG
                docker push $DOCKER_ID/$DOCKER_MOVIE:$DOCKER_TAG
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                cd exam
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install exapp . --values=values.yaml -n dev --create-namespace
                '''
                }
            }

        }
stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                cd exam
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install exapp . --values=values.yaml -n qa --create-namespace
                '''
                }
            }

        }
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                cd exam
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install exapp . --values=values.yaml -n staging --create-namespace
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                cat $KUBECONFIG > .kube/config
                cd exam
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yaml
                helm upgrade --install exapp . --values=values.yaml -n prod --create-namespace
                '''
                }
            }
        }
}
}

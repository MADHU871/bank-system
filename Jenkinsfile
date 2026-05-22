pipeline {

    agent any

    environment {

        IMAGE = "mad0008271/bank-system"
        CONTAINER = "bank-container"

        AZURE_WEBAPP_NAME = "bank-system-webapp"
        AZURE_RESOURCE_GROUP = "bank-group"

    }

    triggers {

        githubPush()

    }

    stages {

        stage('Git Check') {

            steps {

                sh 'pwd'
                sh 'ls -la'

            }
        }

        stage('Docker Build') {

            steps {

                sh '''

                docker build -t $IMAGE:latest .

                '''

            }
        }

        stage('Docker Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''

                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    '''

                }
            }
        }

        stage('Docker Push') {

            steps {

                sh '''

                docker push $IMAGE:latest

                '''

            }
        }

        stage('Docker Pull') {

            steps {

                sh '''

                docker pull $IMAGE:latest

                '''

            }
        }

        stage('Remove Old Container') {

            steps {

                sh '''

                docker stop $CONTAINER || true

                docker rm $CONTAINER || true

                docker container prune -f || true

                '''

            }
        }

        stage('Run Container') {

            steps {

                sh '''

                docker run -d \
                --name $CONTAINER \
                -p 3006:80 \
                $IMAGE:latest

                '''

            }
        }

        stage('Docker Copy') {

            steps {

                sh '''

                docker cp $CONTAINER:/usr/share/nginx/html/index.html .

                '''

            }
        }

        stage('Docker Logs') {

            steps {

                sh '''

                docker logs $CONTAINER

                '''

            }
        }

        stage('Docker Automation Check') {

            steps {

                sh '''

                docker ps

                docker images

                '''

            }
        }

        stage('Azure Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'azure-creds',
                        usernameVariable: 'AZURE_USER',
                        passwordVariable: 'AZURE_PASS'
                    )
                ]) {

                    sh '''

                    az login -u $AZURE_USER -p $AZURE_PASS

                    '''

                }
            }
        }

        stage('Azure Deploy') {

            steps {

                sh '''

                az webapp config container set \
                --name $AZURE_WEBAPP_NAME \
                --resource-group $AZURE_RESOURCE_GROUP \
                --docker-custom-image-name $IMAGE:latest

                '''

            }
        }

        stage('Azure Restart') {

            steps {

                sh '''

                az webapp restart \
                --name $AZURE_WEBAPP_NAME \
                --resource-group $AZURE_RESOURCE_GROUP

                '''

            }
        }

    }

    post {

        success {

            echo 'Pipeline Completed Successfully'

        }

        failure {

            echo 'Pipeline Failed'

        }

        always {

            sh 'docker ps -a'

        }

    }

}
pipeline {

    agent any

    environment {
        SONAR_ORG = "jonpando"
        PROJECT_KEY = "run-aspnetcore-basics"
        IMAGE_NAME = "jonpando26/run-aspnetcore-basics"
    }

    stages {

        stage('Build and QA') {

            parallel {

                stage('Build') {

                    agent {
                        docker {
                            image 'mcr.microsoft.com/dotnet/sdk:8.0'
                            args '--entrypoint="" -u root'
                        }
                    }

                    steps {

                        sh '''
                            dotnet restore AspnetRunBasics.sln

                            dotnet build AspnetRunBasics.sln \
                              --configuration Release
                        '''
                    }
                }

                stage('SonarCloud Analysis') {

                    agent {
                        docker {
                            image 'mcr.microsoft.com/dotnet/sdk:8.0'
                            args '--entrypoint="" -u root'
                        }
                    }

                    steps {

                        withSonarQubeEnv('sonarqube-server') {

                            sh '''
                                dotnet tool install --global dotnet-sonarscanner

                                export PATH="$PATH:/root/.dotnet/tools"

                                dotnet sonarscanner begin \
                                  /k:"$PROJECT_KEY" \
                                  /o:"$SONAR_ORG" \
                                  /d:sonar.host.url="$SONAR_HOST_URL"

                                dotnet build AspnetRunBasics.sln \
                                  --configuration Release

                                dotnet sonarscanner end
                            '''
                        }
                    }
                }
            }
        }

        stage('Docker Build') {

            steps {

                sh '''
                    docker build \
                      -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Publish Docker Image') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                          -u "$DOCKER_USER" \
                          --password-stdin

                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }
    }
}

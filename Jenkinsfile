pipeline {
    agent any

    tools {
        nodejs('Node') 
    }
    
    environment {
        DOCKER_IMAGE_NAME = "kubeanees/hello-world:${BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = "dockercreds"
    }
    
    stages {
        stage('Fetch code') {
            steps {
                git branch: 'main', credentialsId: 'githubcreds', url: 'https://github.com/anees24/hello-world.git'
            }
        }
        stage('Installing Dependencies') {
            steps {
                sh 'npm i install'
            }
        }
        stage('Code Analysis') {
            environment {
                scannerHome = tool 'sonar5'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=hellowworld \
                        -Dsonar.projectName=hellowworld \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=.
                    '''
                }
            }
        }
        stage('Obtaining Results') {
            steps {
                sh 'sleep 10'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE_NAME}")
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE_NAME}").push()
                    }
                }
            }
        }
        stage('Stop Previous Container App') {
            steps {
                sh 'docker ps -f name=kubeanees-hello-world -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -f name=kubeanees-hello-world-v1 -q | xargs -r docker container rm'
            }
        }
        stage('Docker Run App') {
            steps {
                script {
                    sh "docker run -d --name kubeanees-hello-world --rm ${DOCKER_IMAGE_NAME}"
                }
            }
        }
        stage('Delete Old Images with Filter') {
            steps {
                sh 'docker image rm $(docker images --filter "label=name=kubeanees/hello-world" --filter "before=${DOCKER_IMAGE_NAME}" --quiet) -f'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

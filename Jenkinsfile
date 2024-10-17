pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE = "spring_unzip"
        FILE_NAME = "auto_deploy.zip"
        DIR_UNZIP = "auto_deploy"
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CONTAINER = "springboot_jenkins"
    }
    stages {
        stage('Unzip File') {
            steps {
                script {
                    echo "Checking if the file ${FILE_NAME} exists and unzipping it if present..."
                    sh """
                        if [ -f '${FILE_NAME}' ]; then
                            echo "Removing existing files..."
                            rm -rf ${DIR_UNZIP}
                            echo "Unzipping the file..."
                            unzip -o '${FILE_NAME}' -d ${DIR_UNZIP}/
                            echo "Listing files in ${DIR_UNZIP}:"
                            ls -l ${DIR_UNZIP}  # Verify the contents
                        fi
                    """
                }
            }
        }
        stage("Build Application") {
            steps {
                script {
                    echo "Building the application..."
                      sh """
                        if [ -f '${DIR_UNZIP}/pom.xml' ]; then  
                            cd ${DIR_UNZIP}  
                            mvn clean install
                        fi
                    """
                    }
                }
            }
        

        stage("Build Docker Image") {
            steps {
                echo "Building Docker image..."
                dir("${DIR_UNZIP}") { 
                    sh "docker build -t ${DOCKER_IMAGE} ."  
                }
                sh "docker images | grep -i ${IMAGE}"
            }
        }
    }
}

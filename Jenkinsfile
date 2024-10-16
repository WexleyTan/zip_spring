pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        IMAGE = "spring_unzip"
        FILE_NAME = "auto_deploy.zip"
        DIR_UNZIP = "demo"
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CONTAINER = "springboot_jenkins"
        DOCKER_CREDENTIALS_ID = "dockertoken"
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
                        fi
                    """
                }
            }
        }
        stage('Create Dockerfile') {
            steps {
                script {
                    echo "Creating Dockerfile..."
                    '''
                    FROM maven:3.8.7-eclipse-temurin-19 AS build
                    WORKDIR /app
                    COPY . .
                    RUN mvn clean package
                    FROM eclipse-temurin:22.0.1_8-jre-ubi9-minimal
                    COPY --from=build /app/target/*.jar /app/app.jar
                    EXPOSE 9090
                    ENTRYPOINT ["java", "-jar", "app.jar"]
                    '''
                }
            }
        }
        stage('Check Dockerfile') {
            steps {
                script {
                    echo "Checking if Dockerfile exists..."
                    sh """
                        if [ ! -f "${DIR_UNZIP}/Dockerfile" ]; then
                            echo "Dockerfile not found in ${DIR_UNZIP}"
                            exit 1
                        fi
                    """
                }
            }
        }

         stage('Copy Dockerfile') {
            steps {
                script {
                    echo "Copying Dockerfile into demo directory..."
                    sh """
                        if [ -f 'Dockerfile' ]; then
                            cp Dockerfile ${DIR_UNZIP}/
                            echo "Dockerfile copied to ${DIR_UNZIP}"
                        else
                            echo "Dockerfile not found in the root directory!"
                            exit 1
                        fi
                    """
                }
            }
        }

        
       
    }
}

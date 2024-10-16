pipeline {
    agent any
 
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
                        else
                            echo "File ${FILE_NAME} does not exist!"
                            exit 1  # Exit if the file does not exist
                        fi
                    """
                }
            }
        }

        stage('Create Dockerfile') {
            steps {
                script {
                    echo "Creating Dockerfile..."
                    dir("${DIR_UNZIP}") {  
                        sh '''
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
        }
    }
}

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
                        else
                            echo "File ${FILE_NAME} does not exist!"
                        fi
                    """
                }
            }
        }
        
        stage('Create Dockerfile') {
            steps {
                script {
                    echo "Creating Dockerfile..."
                    dir("${DIR_UNZIP}") {  // Ensure the Dockerfile is created in the correct directory
                        sh '''
                            cat <<EOF > Dockerfile
                            FROM maven:3.8.7-eclipse-temurin-19 AS build
                            WORKDIR /app
                            COPY . .
                            RUN mvn clean package

                            FROM eclipse-temurin:22.0.1_8-jre-ubi9-minimal
                            COPY --from=build /app/target/*.jar /app/app.jar
                            EXPOSE 9090
                            ENTRYPOINT ["java", "-jar", "app.jar"]
                            EOF
                        '''
                    
                    }
                }
            }
        }
        
        stage("Clean Package") {
            steps {
                script {
                    echo "Building the application..."
                    dir("${DIR_UNZIP}") {  
                        sh 'mvn clean install' 
                    }
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    echo "Building Docker image..."
                    dir("${DIR_UNZIP}") {
                        sh "docker build -t ${DOCKER_IMAGE} ."
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    echo "Deploying the Docker container..."
                    sh """
                        docker start ${DOCKER_CONTAINER} || docker run --name ${DOCKER_CONTAINER} -d -p 9090:9090 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}

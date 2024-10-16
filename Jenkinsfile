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
    }

    stages {
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

        stage("Build Maven Project") {
            steps {
                script {
                    echo "Building the Maven project..."
                    sh """
                        if [ -f '${DIR_UNZIP}/pom.xml' ]; then  
                            cd ${DIR_UNZIP}
                            mvn clean package
                        fi
                    """
                }
            }
        }
 

        stage("Build Docker Image") {
            steps {
                script {
                    echo "Building Docker image from Dockerfile..."
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    echo "Deploying the Docker container..."
                    sh """
                        docker stop ${DOCKER_CONTAINER} || true
                        docker rm ${DOCKER_CONTAINER} || true
                        docker run --name ${DOCKER_CONTAINER} -d -p 9090:8080 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }
}

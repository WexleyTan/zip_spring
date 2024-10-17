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
                    echo "Checking if the file ${FILE_NAME} exists and unzipping it if present... with job name is ${JOB_NAME}"
                    sh """
                        pwd
                        if [ -f '${FILE_NAME}' ]; then
                            echo "Removing existing files..."
                            rm -rf ${DIR_UNZIP}
                            echo "Unzipping the file..."
                            unzip -o '${FILE_NAME}'
                        fi
                        ls -l
                    """
                }
            }
        }

        stage('Create Dockerfile') {
            steps {
                dir("${DIR_UNZIP}"){
                    sh """
                    echo " # BUILD STAGE
                            
                            # Specify base image for the build stage, which include Maven and JDK 
                            FROM maven:3.8.7-eclipse-temurin-19 AS build
                            
                            # Set the working directory inside the container
                            WORKDIR /app
                            
                            # Copy current local directory to /app which current directory in container
                            COPY . .
                            
                            # Clean the existing build and package the application to create JAR file
                            RUN mvn clean package
                            
                            # RUN STAGE
                            
                            # Specify base image for final stage for running JAVA application
                            #FROM eclipse-temurin:17.0.8_7-jre-alpine
                            FROM eclipse-temurin:22.0.1_8-jre-ubi9-minimal
                            # Copy the executable JAR file from build stage to /app directory in container and rename it to app.jar
                            COPY --from=build /app/target/*.jar /app/app.jar
                            
                            # Expose the port on which your Spring application will run (change as per your application)
                            EXPOSE 8181
                            
                            # Set the command to run your Spring application when the container starts
                            CMD ["java", "-jar", "/app/app.jar"]
                        ENTRYPOINT ["java", "-jar", "app.jar"] " > /var/lib/jenkins/workspace/${JOB_NAME}/${DIR_UNZIP}/Dockerfile
                        ls -l
                    """
                }
            }
        }

        stage("Build Application") {
            steps {
                script {
                    echo "Building the application..."
                    //   sh """
                    //     if [ -f '${DIR_UNZIP}/pom.xml' ]; then  
                    //         cd ${DIR_UNZIP}  
                    //         mvn clean install
                    //     fi
                    // """
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKER_IMAGE} ." 
                    }
                }
            }
        
    }
}

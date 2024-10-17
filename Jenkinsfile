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
                sh """
                echo " FROM maven:3.8.7-eclipse-temurin-19 AS build
                    WORKDIR /app
                    COPY . /app/  
                    RUN mvn clean package
                    FROM eclipse-temurin:22.0.1_8-jre-ubi9-minimal
                    COPY --from=build /app/target/*.jar /app/app.jar
                    EXPOSE 9090
                    ENTRYPOINT ["java", "-jar", "app.jar"] " > /var/lib/jenkins/workspace/${JOB_NAME}/${DIR_UNZIP}/Dockerfile
                    ls -l
                """
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

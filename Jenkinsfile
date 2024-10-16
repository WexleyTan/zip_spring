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
                        fi
                    """
                }
            }
        }
        

        stage('Create Dockerfile') {
            steps {
                script {
                    echo "Creating Dockerfile..."
                    writeFile file: "${DIR_UNZIP}/Dockerfile", text: '''
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

         stage("Build Application") {
            steps {
                script {
                    echo "Building the application..."
                    dir("${DIR_UNZIP}/pom.xml") {
                            sh 'mvn clean install'
                        }
                            error("pom.xml not found in ${DIR_UNZIP}")
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

          


        // stage("Clean Package") {
        //     steps {
        //         script {
        //             echo "Building the application..."
        //             dir("${DIR_UNZIP}") {
        //                 sh 'ls -l'
        //                 sh 'mvn clean install' 
        //             }
        //         }
        //     }
        // }

        // stage("Build Docker Image") {
        //     steps {
        //         echo "Building Docker image..."
        //         dir("${DIR_UNZIP}") { 
        //             sh "docker build -t ${DOCKER_IMAGE} ."  
        //         }
        //         sh "docker images | grep -i ${IMAGE}"
        //     }
        // }

     
    }
}

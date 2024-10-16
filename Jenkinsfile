pipeline {
    agent any 
    environment {
        IMAGE = "neathtan/spring_adv"
        DIR_FILE = "auto_spring"  
        DOCKER_IMAGE = "${IMAGE}:${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockertoken"
        GIT_REPO = "https://github.com/WexleyTan/auto_spring.git"  
        GIT_BRANCH = "master"  
        GIT_MANIFEST_REPO = "https://github.com/WexleyTan/auto_spring_manifest.git"  
        MANIFEST_REPO = "auto_spring_manifest"  
        MANIFEST_FILE_PATH = "deployment.yaml" 
        GIT_CREDENTIALS_ID = 'git_pass' 
    }
    stages {
        stage("Cloning file from GitHub") {
            steps {
                script {
                    echo "Checking if the application repository exists and removing it if necessary..."
                    sh """
                        if [ -d "${DIR_FILE}" ]; then
                            echo "Directory ${DIR_FILE} exists, removing it..."
                            rm -rf ${DIR_FILE}  
                        fi
                    """
                    
                    echo "Cloning the git repository..."
                    sh "git clone -b ${GIT_BRANCH} ${GIT_REPO} ${DIR_FILE}" 
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
        
         stage("Clean Package") {
            steps {
                script {
                    echo "Building the application..."
                    dir("${DIR_FILE}") {  
                        sh 'mvn clean install' 
                    }
                }
            }
        }
    }
}

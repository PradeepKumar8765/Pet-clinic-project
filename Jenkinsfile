pipeline {
    agent {
        docker {
            // Use a Docker image for the build and test environment
            image 'pxdonthala/mavdocim:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    environment {
        SONAR_URL = "http://52.91.55.184:9000" // SonarQube server URL
        DOCKER_IMAGE = "pradeep82kumar/sprint-petclinic:${BUILD_NUMBER}" // Docker image with tag
    }
    stages {
      stage('Checkout') {
        steps {
          sh 'echo passed'
          git branch: 'main', url: 'https://github.com/PradeepKumar8765/Pet-clinic-project.git'
          echo 'checked out'
        }
      }
        stage('Build and Test') {
            steps {
                echo 'Building and testing the project'
                sh 'ls -ltr'
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Static Code Analysis') {
            steps {
                echo 'Running static code analysis'
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.login=$SONAR_AUTH_TOKEN \
                        -Dsonar.host.url=${SONAR_URL}
                    '''
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}"
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    
                    echo "Pushing Docker image: ${DOCKER_IMAGE}"
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "Pet-clinic-project" 
                GIT_USER_NAME = "PradeepKumar8765"  
            }
            steps {
                echo 'Updating Kubernetes deployment file'
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "suhaasq@gmail.com"
                        git config user.name "${GIT_USER_NAME}"
                        
                        # Update the deployment file with the new image tag
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s/deployment.yml
                        
                        # Commit and push the changes
                        git add k8s/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed. Investigating...'
        }
    }
}


pipeline {
  agent {
    docker {
      image 'pxdonthala/mavdocim:latest'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Checkout') {
      steps {
        // Add explicit checkout step
        checkout scm
        sh 'echo passed'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'mvn clean package -DskipTests=true'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://52.91.55.184:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "pradeep82kumar/sprint-petclinic:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
          sh 'docker build -t ${DOCKER_IMAGE} .'
          def dockerImage = docker.image("${DOCKER_IMAGE}")
          docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
            dockerImage.push()
          }
        }
      }
    }
    stage('Update Deployment File') {
      environment {
        GIT_REPO_NAME = "Pet-clinic-project"
        GIT_USER_NAME = "PradeepKumar8765"
        GITHUB_TOKEN = credentials('github')
      }
      steps {
        // Ensure we're in the right directory
        dir("${WORKSPACE}") {
          // Configure Git with safe directory
          sh 'git config --global --add safe.directory "${WORKSPACE}"'
          
          // Configure Git credentials
          sh '''
            git config user.email "suhaasq@gmail.com"
            git config user.name "PradeepKumar8765"
            
            # Ensure we're on the main branch and up to date
            git fetch origin
            git checkout main
            git pull origin main
            
            # Make the changes
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s/deployment.yml
            
            # Stage, commit and push changes
            git add k8s/deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }
  }
}
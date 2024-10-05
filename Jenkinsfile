pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
      options {
        // Clean the workspace before every build
        disableConcurrentBuilds()
        timestamps()
    }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://54.208.148.167:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "murali16394/my-spring-boot-app:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
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
        GIT_REPO_NAME = "CDUsingArgoCD"
        GIT_USER_NAME = "CodeByMurali"
        GIT_REPO_URL = "https://github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git"
    }
    steps {
        script {
            // Clone the repository if it doesn't already exist
            if (!fileExists("CDUsingArgoCD")) {
                sh "git clone ${GIT_REPO_URL}"
            }
        }
        // Change directory to the cloned repository
        dir("${GIT_REPO_NAME}") {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "murali16394@gmail.com"
                    git config user.name "murali.rajendran"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/\(murali16394\/my-spring-boot-app:\)[0-9][0-9]*\b/\1${BUILD_NUMBER}/" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
}
  }
      post {
        always {
            script {
                // Clean the workspace after every build
                echo "Cleaning up the workspace..."
                cleanWs() // This will delete the entire workspace
            }
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

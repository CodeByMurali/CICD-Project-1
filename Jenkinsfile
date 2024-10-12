pipeline {
    agent {
        docker {
            image 'murali16394/jenkins-docker-custom-agent-1:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
        }
    }
    
    options {
        // Clean the workspace before every build
        disableConcurrentBuilds()
        timestamps()
    }
    
    parameters {
        string(name: 'CICDSERVER_IP', defaultValue: '54.147.46.158', description: 'Enter the SonarQube server IP address')
    }

    environment {
        SONAR_URL = "http://${params.CICDSERVER_IP}:9000" // Define SonarQube URL here
        DOCKER_IMAGE = "murali16394/my-spring-boot-app:${BUILD_NUMBER}" // Docker image name with version
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
                sh 'mvn clean package'
            }
        }
        // SonarQube server pre-installed
        stage('SonarQube Initialization and Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }
        // Build Docker Image
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }
        // Trivy pre-installed - Scan for Docker Image
        stage('Trivy Scan Docker Image') {
            steps {
                sh 'echo $PATH' // Debug the PATH variable
                sh '/usr/local/bin/trivy --version'

                // Capture the exit status of the Trivy scan
                script {
                    def trivyScanStatus = sh(script: '/usr/local/bin/trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}', returnStatus: true)

                    // Check the exit status and log a warning if vulnerabilities are found
                    if (trivyScanStatus != 0) {
                        echo 'Warning: Security vulnerabilities found in the Docker image!'
                    } else {
                        echo 'No security vulnerabilities found in the Docker image.'
                    }
                }
            }
        }
        stage('Docker push by retrieving credentials from vault') {
            steps {
                withVault(configuration: [timeout: 60, vaultCredentialId: 'vault-token', vaultUrl: "http://${params.CICDSERVER_IP}:8200"], 
                        vaultSecrets: [[path: 'secret/dockerhub', secretValues: [[vaultKey: 'username', envVar: 'DOCKER_USERNAME'], [vaultKey: 'password', envVar: 'DOCKER_PASSWORD']]]]) {
                    script {
                        // Log in to Docker
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        echo "Docker hub login successful via vault"
                        // Define the Docker image and push it
                        def dockerImage = docker.image("${DOCKER_IMAGE}")
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        // Commented: Push Docker Image to Docker Hub using Jenkins Docker plugin credentials
        // stage('Push Docker Image') {
        //     environment {
        //         DOCKER_IMAGE = "murali16394/my-spring-boot-app:${BUILD_NUMBER}"
        //         REGISTRY_CREDENTIALS = credentials('docker-cred')
        //     }
        //     steps {
        //         script {
        //             def dockerImage = docker.image("${DOCKER_IMAGE}")
        //             docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
        //                 dockerImage.push()
        //             }
        //         }
        //     }
        // }

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
                            sed -i "s/murali16394/my-spring-boot-app:[0-9][0-9]*/murali16394/my-spring-boot-app:${BUILD_NUMBER}/" spring-boot-app-manifests/deployment.yml
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
                cleanWs()
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
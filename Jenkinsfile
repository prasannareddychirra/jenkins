pipeline {
  agent {
    docker {
      image 'prasannareddychirra/jenkins-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        git branch: 'main', url: 'https://github.com/prasannareddychirra/jenkins.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'mvn clean package'
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "prasannareddychirra/taxi-cab-application:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker_login')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker_login") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "jenkins"
            GIT_USER_NAME = "prasannareddychirra"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "prasannareddy.chirra@gmail.com"
                    git config user.name "Prasanna Kumar Reddy"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" argocd/deployment.yml
                    git add argocd/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
}
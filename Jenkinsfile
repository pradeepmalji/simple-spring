pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
    }

  }
  stages {
    stage('Build') {
      steps {
        container(name: 'maven') {
          sh 'mvn clean install'
        }

      }
    }

    stage('Load') {
      steps {
        script {
          app = docker.build("${IMAGE_TAG}")
        }

      }
    }

    stage('Deploy') {
      steps {
        script {
          docker.withRegistry( "https://registry.hub.docker.com", registryCredential ) {
            // dockerImage.push()
            app.push("${IMAGE_TAG}")
          }
        }

      }
    }

    stage('DeployToGCP') {
      steps {
        container(name: 'kubectl') {
          sh 'kubectl apply -f sample.yaml'
        }

      }
    }

  }
  environment {
    PROJECT = 'pradeepmalji'
    APP_NAME = 'gceme'
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = 'jenkins-cd'
    CLUSTER_ZONE = 'us-east1-d'
    IMAGE_TAG = "docker.io/${PROJECT}/${APP_NAME}:latest"
    JENKINS_CRED = "${PROJECT}"
    registryCredential = 'docker_hub'
  }
}
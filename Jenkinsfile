pipeline {
  environment {
    PROJECT = "codepro-b61b8"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "us-east1-d"
    IMAGE_TAG = "gcr.io/${PROJECT}/${APP_NAME}:latest"
    JENKINS_CRED = "${PROJECT}"
    registryCredential = "docker_hub"
  }
  
  agent {
    kubernetes {
      label 'sample-app'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: maven
    image: maven:3-alpine
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }
 
  stages {
    stage('Build') {
      steps{
        container('maven') {
          sh 'mvn clean install'
        }
      }
    }
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG} ."
        }
      }
    }
    
    stage('Deploy') {
      steps{
        script { 
          docker.withRegistry( "https://registry.hub.docker.com", registryCredential ) {
           // dockerImage.push()
          app.push("${IMAGE_TAG}")
          }
        }
      }
    }
    stage('DeployToGCP'){
      steps{
        container('kubectl'){
            sh 'kubectl apply -f sample.yaml'
        }  
      }
    }
  }
}

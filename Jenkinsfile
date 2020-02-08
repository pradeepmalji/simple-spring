pipeline {
  environment {
    PROJECT = "pradeepmalji"
    APP_NAME = "gceme"
    FE_SVC_NAME = "${APP_NAME}-frontend"
    CLUSTER = "jenkins-cd"
    CLUSTER_ZONE = "us-east1-d"
    IMAGE_TAG = "docker.io/${PROJECT}/${APP_NAME}:latest"
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
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
    resources:
      limits:
        cpu: 1
        memory: 2Gi
      requests:
        cpu: 1
        memory: 2Gi
    command:
    - cat
    tty: true
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: dind
    image: docker:18.05-dind
    securityContext:
      privileged: true
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
  volumes:
      - name: dind-storage
        emptyDir: {}
"""
}
  }
  stages {
    stage(‘Build’) {
      steps{
        container('maven') {
          sh 'mvn clean install'
        }
      }
    }
    stage(‘Load’) {
      steps{
        container('maven') { {
          app = docker.build("${IMAGE_TAG}")
        }
      }
    }
    stage(‘Deploy’) {
      steps{
        container('maven') { {
          docker.withRegistry( "https://registry.hub.docker.com", registryCredential ) {
           // dockerImage.push()
          app.push("${IMAGE_TAG}")
          }
        }
      }
    }
    stage(‘DeployToGCP’){
      steps{
        container('kubectl'){
            sh 'kubectl apply -f sample.yaml'
        }  
      }
    }
  }
}

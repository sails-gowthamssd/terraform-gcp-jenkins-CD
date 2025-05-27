pipeline {
  agent any

  environment {
    GOOGLE_APPLICATION_CREDENTIALS = "${WORKSPACE}\\terraform-sa.json"
    PROJECT_ID = "my-kubernetes-project-456905"
    CLUSTER_NAME = "hello-jenkins-gke"
    REGION = "us-central1-b"
    K8S_NAMESPACE = "default"
  }

  stages {
    stage('Checkout Helm Charts') {
      steps {
        git url: 'https://github.com/sails-gowthamssd/terraform-gcp-jenkins-CD.git', branch: 'main'
      }
    }

    stage('Prepare Credentials') {
      steps {
        withCredentials([file(credentialsId: 'TERRAFORM_SA_KEY', variable: 'TF_SA_KEY')]) {
          bat 'copy %TF_SA_KEY% terraform-sa.json'
        }
      }
    }

    stage('gcloud Auth & kubectl Setup') {
      steps {
        bat 'gcloud auth activate-service-account --key-file=terraform-sa.json'
        bat 'gcloud config set project %PROJECT_ID%'
        bat 'gcloud container clusters get-credentials %CLUSTER_NAME% --region %REGION%'
        bat 'kubectl config set-context --current --namespace=%K8S_NAMESPACE%'
      }
    }

    stage('Helm Deploy') {
      steps {
        dir('helm/hello-world') {
          bat 'helm upgrade --install hello-world . --namespace %K8S_NAMESPACE%'
        }
      }
    }
  }

  post {
    cleanup {
      bat 'del terraform-sa.json'
    }
  }
}

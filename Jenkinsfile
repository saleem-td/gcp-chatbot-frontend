pipeline {
  agent any
  environment {
    PROJECT_ID = "gcp-chatbot-proj"
    REGION = "europe-west9"
    REPO = "frontend-repo"
    CLUSTER = "frontend"
    CLUSTER_REGION = "europe-west9"
    K8S_DIR = "frontend"
  }
  
  stages {
    stage('Checkout') {
      steps {
        withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_TOKEN')]) {
          git branch: 'main',
              url: "https://$GITHUB_TOKEN@github.com/saleem-td/gcp-chatbot-frontend.git"
        }
      }
    }

    stage('GCP Auth') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GCP_SA_KEY')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GCP_SA_KEY --project=$PROJECT_ID
            gcloud config set project $PROJECT_ID
            gcloud auth configure-docker ${REGION}-docker.pkg.dev -q
          '''
        }
      }
    }

    stage('Build & Push Image') {
      steps {
        sh '''
          TAG=$(date +%Y%m%d%H%M%S)
          IMAGE=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/streamlit:${TAG}
          echo $IMAGE > image.txt
          docker build -t $IMAGE ./frontend
          docker push $IMAGE
        '''
      }
    }

    stage('Deploy to GKE') {
      steps {
        sh '''
          IMAGE=$(cat image.txt)
          gcloud container clusters get-credentials $CLUSTER --region $CLUSTER_REGION --project $PROJECT_ID
          sed "s|IMAGE_PLACEHOLDER|$IMAGE|g" ${K8S_DIR}/k8s-frontend.yaml > /tmp/deployment.yaml
          kubectl apply -f /tmp/deployment.yaml
          kubectl rollout status deployment/chatbot-frontend --timeout=180s
        '''
      }
    }
  }
}

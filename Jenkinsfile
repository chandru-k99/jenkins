pipeline {
    agent any

    environment {
        GCP_PROJECT_ID = credentials('GCP_PROJECT_ID')
        TF_STATE_BUCKET_NAME = credentials('GCP_TF_STATE_BUCKET')
        IMAGE_TAG = "${env.GITHUB_SHA}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Authenticate to Google Cloud') {
            steps {
                script {
                    def serviceAccount = 'tf-gke-test@' + env.GCP_PROJECT_ID + '.iam.gserviceaccount.com'
                    withCredentials([string(credentialsId: 'GOOGLE_APPLICATION_CREDENTIALS', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                        sh "gcloud auth activate-service-account --key-file=${GOOGLE_APPLICATION_CREDENTIALS}"
                    }
                }
            }
        }

        stage('Build and push Docker image') {
            steps {
                dir('nodeapp') {
                    sh "docker build -t us.gcr.io/${env.GCP_PROJECT_ID}/nodeappimage:${IMAGE_TAG} ."
                    sh "docker push us.gcr.io/${env.GCP_PROJECT_ID}/nodeappimage:${IMAGE_TAG}"
                }
            }
        }

        stage('Setup Terraform') {
            steps {
                sh "terraform init -backend-config=\"bucket=${TF_STATE_BUCKET_NAME}\" -backend-config=\"prefix=test\""
            }
        }

        stage('Terraform Plan') {
            steps {
                sh "terraform plan -var=\"region=us-central1\" -var=\"project_id=${env.GCP_PROJECT_ID}\" -var=\"container_image=us.gcr.io/${env.GCP_PROJECT_ID}/nodeappimage:${IMAGE_TAG}\" -out=PLAN"
            }
        }

        stage('Terraform Apply') {
            steps {
                sh "terraform apply PLAN"
            }
        }
    }
}

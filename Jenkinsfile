#!/usr/bin/env groovy
pipeline {
    agent any
    environment {
        GKE_GCLOUD_AUTH_PLUGIN = '/var/lib/jenkins/google-cloud-sdk/bin/gke-gcloud-auth-plugin'
        GCP_SERVICE_ACCOUNT = credentials('gcp-service-account')
        PROJECT_ID = credentials('my-gcp-project-id')
        ZONE = 'us-central1'
        TOKEN = "/var/lib/jenkins/google-cloud-sdk/bin/.kube/gke_gcloud_auth_plugin_cache"
    }
    stages {
        stage('Install Google Cloud SDK') {
            steps {
                sh 'curl https://sdk.cloud.google.com | bash'
                sh 'exec -l $SHELL'
                sh 'gcloud config set disable_usage_reporting false'
                sh 'gcloud init'
            }
        }
        stage("Create a GCP Cluster") {
            steps {
                script {
                    dir('terraform') {
                        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_CREDENTIALS')]) {
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud auth activate-service-account --key-file=$GOOGLE_CREDENTIALS'
                            sh "terraform init"
                            sh "terraform apply -auto-approve"
                        }
                    }
                }
            }
        }
        stage("Install Plugins for Cluster Connection") {
            steps {
                script {
                    dir('kubernetes') {
                        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_CREDENTIALS')]) {
                            sh 'export PATH="$PATH:/var/lib/jenkins/google-cloud-sdk/bin/"'  // Add this line to include the gcloud binary directory in PATH
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud auth activate-service-account --key-file=$GOOGLE_CREDENTIALS'
                            //sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud components remove gke-gcloud-auth-plugin'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud components install gke-gcloud-auth-plugin'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud components install kubectl'
                            sh 'echo "export USE_GKE_GCLOUD_AUTH_PLUGIN=True" >> ~/.bashrc'
                            sh 'source ~/.bashrc'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud components update'
                        }
                    }
                }
            }
        }
        stage("Gke-Auth-and-Deployment") {
            steps {
                script {
                    dir('kubernetes') {
                        withCredentials([file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_CREDENTIALS')]) {
                            sh 'export PATH="$PATH:/var/lib/jenkins/google-cloud-sdk/bin/"'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud auth activate-service-account --key-file=$GOOGLE_CREDENTIALS'
                            //sh '/var/lib/jenkins/google-cloud-sdk/bin/" gke-gcloud-auth-plugin --impersonate_service_account string"'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/gcloud container clusters get-credentials ayomide-cluster-live --region us-central1 --project orbital-avatar-378601'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply -f nginx-deployment.yaml'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply -f nginx-service.yaml'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply -f microservices-demo-itsmide/deploy/kubernetes/complete-demo.yaml'
                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply -f sock-shop.yaml'
                            dir('microservices-demo-itsmide') {
                                dir('deploy') {
                                    dir('kubernetes') {
                                        dir('manifests-monitoring') {
                                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl create -f 00-monitoring-ns.yaml'
                                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply $(ls *-prometheus-*.yaml | awk ' { print " -f " $1 } ')'
                                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply $(ls *-grafana-*.yaml | awk ' { print " -f " $1 }'  | grep -v grafana-import)'
                                            sh '/var/lib/jenkins/google-cloud-sdk/bin/kubectl apply -f 23-grafana-import-dash-batch.yaml'
                                        }
                                    }
                                }
                            }                       
                        }
                    }
                }
            }
        }
    }
}

# Installing Argocd on GKE via git actions  

This documentation is a walk-through for installing argocd on Google Kubernetes Engine (GKE) via git actions.  

## The workflow  
- Setup the GKE cluster  
- Setup git secrets  
- Setup the git actions  
- Deploy the app on the argocd UI

## Setup the GKE cluster  
The GKE cluster is created on the google cloud platform console. 

## Setup the git secrets  
The git repository secrets are created with the google service account key and google cloud project ID.  

## Setup the git actions  
```
name: Deploy and Configure ArgoCD in GKE

on:
  push:
    branches:
    - main

env:
  GKE_KEY: ${{ secrets.GKE_KA_KEY }}  # TODO: Update with mavencode key
  PROJECT_ID: ${{ secrets.GKE_PROJECT }}  # TODO: Update with mavencode project key
  GKE_CLUSTER: gke-cluster    # TODO: update with mavencode GKE cluster name
  GKE_ZONE: us-central1-c     # TODO: update with mavencode GKE cluster zone

jobs:
  setup-install:
    name: Setup, Install
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2
    
    - name: Set up Google Cloud CLI
      uses: google-github-actions/setup-gcloud@94337306dda8180d967a56932ceb4ddcf01edae7
      with: 
        service_account_key: ${{ secrets.GKE_KEY }}
        project_id: ${{ secrets.GKE_PROJECT }}

    - name: Setup GKE credentials
      uses: google-github-actions/get-gke-credentials@v0
      with:
        cluster_name: ${{ env.GKE_CLUSTER }}
        location: ${{ env.GKE_ZONE }}
        credentials: ${{ secrets.GKE_KEY }}
        
    - name: Install argocd and exposed argocd-server
      run: |-   
        kubectl delete namespace argocd
        
        kubectl create namespace argocd
        
        kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

        kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'  
```  
## Deploy the app from the argocd UI
The argocd resources are created on the GKE. Login into your gcp account to access the resources in the cluster.  

![created services](https://github.com/kennedyuche/doc-images/blob/main/argocd8.PNG)  

The argocd UI is accessed by clicking on the endpoint IP address of the argocd-server.  

To login into the argocd UI, you will need the default password of the system.  

You get this password using the command below on the gcp cloud shell:  

```  
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d  
```  
![argocd password](https://github.com/kennedyuche/doc-images/blob/main/argocd12.PNG)  

The default password is highlighted in the image above (`b9tLM6cE-3XsXK93`).  

username: `admin`
password: `b9tLM6cE-3XsXK93`  (use the password derived from the above step)

You can update the default password.  

Login to the argocd UI and deploy the application.
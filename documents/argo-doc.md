# Installing Argocd on GKE via git actions  

This documentation is a walk-through for installing argocd on Google Kubernetes Engine (GKE) via git actions.  

## The workflow  
- Setup the GKE cluster  
- Setup git secrets  
- Setup the git actions  
- Deploy the app on the argocd UI

## Setup the GKE cluster  
Create the GKE cluster from the google cloud platform console. 

## Setup the git secrets  
Setup git repository secrets with the google service account key and google cloud project ID.  

## Setup the git actions  
```
name: Deploy and Configure ArgoCD in GKE    # Give the git actions a suitable name

on:                                         # The action triggers when you push to the main branch
  push:           
    branches:
    - main

env:
  GKE_KEY: ${{ secrets.GKE_KA_KEY }}        # TODO: Update the with the name of the git secret you created using the gcp service key
  PROJECT_ID: ${{ secrets.GKE_PROJECT }}    # TODO: Update with the name of the git secret you created using the project ID
  GKE_CLUSTER: ${{ secrets.GKE_CLUSTER }}   # TODO: update with the name of the git secret you created using the gke cluster name
  GKE_ZONE: ${{ secrets.GKE_ZONE }}         # TODO: update with the name of the git secret you created using the gke cluster zone

jobs:
  setup-install:                            # Name the job to be performed by the git actions
    name: Setup, Install
    runs-on: ubuntu-latest

    steps:                                  # This checkouts the git repository you are working with
    - name: Checkout Repository
      uses: actions/checkout@v2
    
    - name: Set up Google Cloud CLI         # This sets up the Google Cloud CLI with the secret keys defined above
      uses: google-github-actions/setup-gcloud@94337306dda8180d967a56932ceb4ddcf01edae7
      with: 
        service_account_key: ${{ secrets.GKE_KEY }}
        project_id: ${{ secrets.GKE_PROJECT }}

    - name: Setup GKE credentials           # This sets up the working GKE cluster with the cluster credentials defined above
      uses: google-github-actions/get-gke-credentials@v0
      with:
        cluster_name: ${{ secrets.GKE_CLUSTER }}
        location: ${{ secrets.GKE_ZONE }}
        credentials: ${{ secrets.GKE_KEY }}
        
    - name: Install argocd                  # This installs the argocd on the GKE cluster established above
      run: |-     
        kubectl create namespace argocd     
        
        kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml  
```  

## Deploy the app from the argocd UI  
Once the git actions is triggered, the jobs outlined above will be exected on your GCP GKE cluster.  

Login to your GCP account and navigate to the GKE cluster to view your deployment.  

The services & ingress section of the kubernetes engine will be similar with the image below.  

![Services & Ingress](https://github.com/MavenCode/argocd-manifest/blob/argocd-image/argocd-images/argocd%201.PNG)  

To deploy an application with argocd, you need to access the UI (user interface).  

To be able to view the argocd UI, you need to expose the argocd-server.  

The argocd-server is exposed by running the following command from the Google Cloud Shell terminal:  

```  
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'  
```  

Refreshing the Kubernetes engine page, the Services & Ingress section updates to an output similar to the image below.  

![Updated Services & Ingress](https://github.com/MavenCode/argocd-manifest/blob/argocd-image/argocd-images/argocd%202.PNG)


The argocd UI is accessed by clicking on the endpoint IP address of the argocd-server.  

To login into the argocd UI, you will need the default password of the system.  

You get this password using the command below on the gcp cloud shell:  

```  
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d  
```  

![argocd password](https://github.com/MavenCode/argocd-manifest/blob/argocd-image/argocd-images/argocd12.PNG)  

The default password is highlighted in the image above (`b9tLM6cE-3XsXK93`).  

username: `admin`
password: `b9tLM6cE-3XsXK93`  (use the password you derived from the step above)

You can update the default password.  

Login to the argocd UI and deploy the application.
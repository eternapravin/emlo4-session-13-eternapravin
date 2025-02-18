# emlo4-session-13-eternapravin
Session13 - Kubernetes Introduction

### Contents
  - [Requirements]
  - [Step-by-step-Approach-in-Development]
  - [Observation]
  - [Results]

### Requirements
1. Create a Minikube Cluster Locally or with a t3a.medium Instance on EC2
2. Deploy the CatDog/Dog Breed Classifier model with FastAPI, use FastHTML for the frontend
   - Your Deployment must have 2 Pods for the Model Service
   - It should have a Service and an Ingress
3. What to Submit
    - Github Repo with Deployment YAML Files
    - Instructions to
      - Start MiniKube
      - Create the Deployment
      - Tunnel to the Ingress
      - Access the FastAPI docs page
    - Output of the following in a .md file in your repository
      - kubectl describe <your_deployment>
      - kubectl describe <your_pod>
      - kubectl describe <your_ingress>
      - kubectl top pod
      - kubectl top node
      - kubectl get all -o yaml

# Personio SRE Challenge

## Introduction
In this challenge i tried to present a simple solution of my daily work, those are the technologies in the stack i currently work, i am only missing Helm because i think it was not necessary. 
### Used technologies
- Terraform
- CircleCI
- Google Kubernetes Engine
- Docker
- Golang

## Terraform
The terraform code is in the following [repository](https://github.com/ivantrips/personio-terraform) and it is using a GCP account in the free tier so feel free to try the code with
```
terraform init
terraform plan
terraform apply
```
As a note i attached a service account with the permissions needed to create the cluster in GCP

The terraform repository has three files but the important one is the one named **cluster.tf** which has a simple cluster with one node pool using preemtible machines, which are way more cheaper than on premise and are really good for testing or stateless applications

## CircleCI
The CircleCI build has three components
* test
* build
* deploy

This is more detailed in the `.circleci/config.yml` and basically it will build a Dockerfile, push it to one of my public repositories and then apply the configuration in `deploy/k8s/` replacing the image tag with the current build, this is filtered to only work on the master branch.

## GKE
### Deployment
My deployment has some environment variables related to the application but it is not using secrets, configmap or volumes.
Which is important is to have a `readinessProbe` so Kubernetes is able to LoadBalance the request between the pods and avoid any 503 response code
### HPA
This application is going to scale based in CPU having maximum 10 replicas, i choose CPU because the application is not using memory for any process it is just returning a simple json response which is more CPU consuming and i have 2 replicas as minimun, because it is a really small binary that can take requests in less than 1 seconds.
### Service
I choose to use a Loadbalancer because i didn't need a domain or anything for this test just wanted to be exposed to the internet and this was the easiest way

## Docker
I created a simple Alpine image to store my Golang binary and it contains a entrypoint executing that web server

## Golang
I choosed Golang to make this simple application, because binaries tend to be more easy to manage in Kubernetes because of the startup time and the size of the Docker images, and having a stateless application usually that matters the most to scale horizontally

# Improvements
If i could have time to do a more production ready setup i would choose to
- Store secrets using vault
- Have separate environments on the CI/CD pipeline (qa, staging, production)
- Add a Pod Disruption budget according to the rules needed
- Add taints and better tags for the application
- Use helm if i am going to need to use similar images in different environments/clients
- Store the terraform state anywhere to be able to share it with my coworkers
- Use multiregional cluster or choose a better zone depending on the use of the application


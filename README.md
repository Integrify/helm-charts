# Integrify Helm Charts Repository

## Summary
This repository holds the helm chart for deploying Integrify into a Kubernetes cluster. 

Currently, we provide base values files to deploy to [Minikube](https://minikube.sigs.k8s.io/docs/) and [Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html) clusters. These charts are a basic starting point for deployment. You will need to modify these files for your specific environment. We will add more values files as they become available.

**_Please note: Minikube is designed to be an environment for testing Kubernetes applications. We do not recommend, nor do we support, production installations of Integrify in Minikube._**

## Setting Up Kubernetes
Support for installing, administering, optimizing, securing, or maintaining your Kubernetes installation is provided by certified Integrify partners. If you are interested in working with one of our partners, please contact us at customer-service@integrify.com. Customers wishing to manage the installation themselves can deploy Kubernetes based on their company's best practices and policies.

## Before You Begin
- You will need to download the proper environment (minikube or EKS) values.yaml template file (available from this repo) and update it with your Integrify installation information prior to deploying the application.
- In addition, you will need to open a support request via [Integrify Support](https://support.integrify.com) to get AWS credentials to pull the Integrify services images. 
- We will also provide you with the SQL scripts needed to setup your Integrify databases in the aforementioned support request.

## General Prerequisites

- Microsoft SQL Server (2016+)
  - We recommend two separate databases. A consumers database which will hold application and instance configuration data and a data specific database.
- Mongo (3.4+)
- Redis
- Kubernetes (1.23+)
- Helm (3.7+)

## Quick Setup

To add the Integrify repo...
```
helm repo add integrify https://integrify.github.io/helm-charts
```

Once the repo has been added and your values.yaml file has been updated you can install with...
```
helm install -f <values.yaml> integrify integrify/integrify
```



# Integrify Helm Charts Repository

This repository will hold the helm charts to deploy Integrify. Currently we have separate charts for different environments but these will eventually be merged into one primary chart.



## TL;DR
You will need to download the values.yaml template available on this page and update it with your Integrify installation information prior to installing the application.

To add the Integrify repo...
```
helm repo add integrify https://integrify.github.io/helm-charts
```

Once the repo has been added and your values.yaml file has been updated you can install with...
```
helm install -f <values.yaml> integrify <chart>
```

## Before You Begin

### Prerequisites

- Microsoft SQL Server (2016+)
- Mongo (3.4+)
- Redis
- Kubernetes (1.23+)
- Helm (3.7+)

### Setting Up Kubernetes
We currently provide charts for deployment to Minikube and Amazon Web Services Elastic Kubernetes Servive (EKS). READMEs for deployment on each chart can be found in their respective chart above. *Please note: Minikube is designed to be a environment for testing Kubernetes applications including Integrify. We do not recommend, nor do we support, production installations of Integrify in Minikube.*
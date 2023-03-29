# Integrify Helm Charts Repository

This repository will hold the helm charts to deploy Integrify. Currently we have separate charts for different Kubernetes environments but these will eventually be merged into one primary chart.



## Summary
- You will need to download the proper environment (minikube or EKS) values.yaml template available from this page and update it with your Integrify installation information prior to installing the application.
- You will need to contact [Integrify Support](https://support.integrify.com) to get AWS credentials to pull the Integrify images. If you have an AWS account you can use an IAM identity within your account but you will still need to contact Integrify so we can provide you the proper IAM Role ARN to assume.

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
We currently provide charts for deployment to Minikube and Amazon Web Services Elastic Kubernetes Servive (EKS). READMEs for deployment on each chart can be found in their respective chart above. **_Please note: Minikube is designed to be an environment for testing Kubernetes applications including Integrify. We do not recommend, nor do we support, production installations of Integrify in Minikube._**

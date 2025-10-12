# Nutrient Workflow Automation (Formerly Integrify) Helm Charts Repository

## Summary
This repository holds the helm chart for deploying Nutrient Workflow Automation into a [Kubernetes](https://kubernetes.io) cluster. 

## Setting Up Kubernetes
Support for installing, administering, optimizing, securing, or maintaining your Kubernetes installation is provided by certified Nutrient partners. If you are interested in working with one of our partners, please contact us at customer-success@integrify.com. 

Customers wishing to manage the installation themselves can deploy Kubernetes based on their company's best practices and policies. 

## Before You Begin
- You will need to download the [values.yaml](values.yaml) template and update it with your Nutrient Workflow Automation installation information prior to deploying the application.
- In addition, you will need to open a [Help Ticket](https://support.integrify.com/workflow/actions/request/entry/246) via [Integrify Support](https://support.integrify.com) to obtain AWS credentials to pull the needed Nutrient Workflow Automation service images. 
- We will also provide you with the SQL scripts needed to setup your Integrify databases in the aforementioned support request.

## General Prerequisites

- Kubernetes (1.23+)
- Helm (3.7+)

## Data Store Prerequisites:
The following data store prerequisites are NOT included in the Workflow Automation Helm charts: 

- Microsoft SQL Server (2016+)
  - We recommend two separate databases. A consumers database which will hold application and instance configuration data and a data specific database.
- Mongo (3.4+)
- Redis
- AWS Credentials with access to S3 or a Kubernetes Persistent Volume and Persistent Volume Claim for file storage.
- PostgreSQL - This supports features related to document previewing, editing, authoring, etc. The chart does not provide direct installation of PostgreSQL. However, the chart does suggest generation of a PostgreSQL cluster via the [CloudNativePG](https://cloudnative-pg.io/) operator.

## Quick Setup

To add the Nutrient Workflow Automation repo...
```
helm repo add workflow-automation https://integrify.github.io/helm-charts
```

Once the repo has been added and your values.yaml file has been updated you can install with...
```
helm install -f <values.yaml> workflow-automation workflow-automation/integrify
```

## Sample values

* [Default](values.yaml)
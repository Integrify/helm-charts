# helm-charts
# Repository for Integrify Helm Charts

This repository will hold the helm charts to deploy Integrify. Currently we have separate charts for different environments but these will eventually be merged into one chart.

Prior to installation you will need to download the values.yaml template from this page and update it with your installation specific information.

To add the repo...
```
helm repo add integrify https://integrify.github.io/helm-charts
```

Once the repo has been added and your values.yaml file has been updated you can install with...
```
helm install -f <values.yaml> integrify <chart>
```

# Setup Integrify Locally On Minikube

### Prerequisites
  - MSSQL Server (2016+)
  - Mongo (3.4+)
  - Redis
  - Minikube
    - **NOTE** If you plan on running Minikube on Docker, it should be run natively on docker engine on Linux. You may be able to get this working on Docker Desktop using the `minikube proxy` command after application installation to open the ingress to your local host machine but that has not yet been tested. This is due to Docker Desktop running all containers in it's own small linux container.
  - Helm (3.7+)
  - An AWS Account with an IAM user that has permissions to assume a role provided by Integrify which will allow you to pull the deployment images

**_Please note: Minikube is designed to be an environment for testing Kubernetes applications including Integrify. We do not recommend, nor do we support, production installations of Integrify in Minikube._**


### Install
**1. Setup Minikube And Environment**
- Create a directory where you can store Integrify attachment, report, and temp files.
```
mkdir integrify-demo
```

- CD into the directory created above and create a files directory.
```
cd integrify-demo && mkdir files
```

- Start Minikube mounting the files directory.
```
minikube start --mount --mount-string="$PWD/files:/host/files" --driver=docker
```

- Get the ip address of your minikube container
```
minikube ip
```

- Update your /etc/hosts file with this ip address and the url you'll be using to access Integrify. For example...
```
192.168.49.2  integrify.demo
```

- If you want to run the minikube dashboard, run the following in a separate terminal window. This terminal window will need to stay running
```
minikube dashboard
```

- Enable the minikube dashboard metrics-server for metrics reporting. This is optional.
```
minikube addons enable metrics-server
```

- Enable the needed minikube addons. The registry-creds addons will allow you to add AWS creds for image pulls while the ingress creates the NGINX ingress controller to create an ingress load-balancer.
```
minikube addons enable registry-creds
```
```
minikube addons enable ingress
```

- Configure the registry-creds addon with your AWS credentials to be able to pull the images from AWS ECR
```
minikube addons configure registry-creds
```

**In the registry-creds configuration follow the prompts to enable AWS Elastic Container Registry. You'll need to supply the AWS Access Key ID and AWS Secret Access Key from your AWS Account IAM user, the AWS Region (us-east-1), and the Integrify Account ID and Role ARN provided by Integrify**

**You won't be able to pull images from ECR until you see an awsecr-cred secret in the default namespace. This can take a few minutes**

**2. Install Integrify**

- Download the default [values.yaml](https://github.com/Integrify/helm-charts/blob/main/values-minikube.yaml) template file for Minikube from the Integrify Helm Charts GitHub page and save it in the directory you created in step 1

- Update the downloaded values.yaml template file with your Integrify installation details

- Add the Integrify Helm repo
```
helm repo add integrify https://integrify.github.io/helm-charts
```

- Deploy release called integrify via helm. Make sure the `-f` flag is pointing to your values.yml file
```
helm install -f values.yaml integrify integrify/integrify-minikube
```

- It may take awhile to pull all the images and start the pods. If you get any sort of image pull error check the pod for the specific error. If it's a timeout error delete the pod and let it restart. 

- Once all pods are running you should be able to access your instance by opening a browser and going to http://integrify.demo
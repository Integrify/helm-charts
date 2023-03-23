# Setup Integrify Cloud Locally On Minikube

### Prerequisites
  - MSSQL Server (2016+)
  - Mongo (3.4+)
  - Redis
  - Minikube
    - **NOTE** If you plan on running Minikube on Docker, it should be run natively on docker engine on Linux. You may be able to get this working on Docker Desktop using the `minikube proxy` command after application installation to open the ingress to your local host machine but that has not yet been tested. This is due to Docker Desktop running all containers in it's own small linux container.
  - Kubectl
  - Helm
  - An AWS Account with an IAM user that has permissions to assume a role provided by Integrify which will allow you to pull the deployment images


### Install
**1. Setup Minikube And Environment**
- CD into the helm-minikube directory
```
cd helm-minikube
```

- Create a files/config directory. The files directory will store attachments, reports, and other files needed by Integrify. It is ignored by Git.
```
mkdir -p files/config
```

- Start Minikube
```
minikube start --mount --mount-string="$PWD/files:/host/files" --driver=docker
```
*If you already have a minikube instance running or want to name the minikube instance add a ```-p``` to the above command..*
```
minikube start -p {instance_name} --mount --mount-string="$PWD/files:/host/files" --driver=docker
```

- Get the ip address of your minikube container
```
minikube ip
```

- Update your /etc/hosts file with this ip address and the url you'll be using to access Integrify. For example...
```
192.168.49.2  integrify.demo
```

- Enable the needed minikube addons
```
minikube addons enable metrics-server
```
```
minikube addons enable registry-creds
```
```
minikube addons enable ingress
```

- If you want to run the minikube dashboard run the following in a separate terminal window. This terminal window will need to stay running
```
minikube dashboard
```

- Configure the registry-creds addon with your AWS credentials to be able to pull the images from AWS ECR
```
minikube addons configure registry-creds
```

**In the registry-creds configuration follow the prompts to enable AWS Elastic Container Registry. You'll need to supply the AWS Access Key ID and AWS Secret Access Key from your AWS Account IAM user, the AWS Region (us-east-1), and the Integrify Account ID and Role ARN provided by Integrify**

**You won't be able to pull images from ECR until you see an awsecr-cred secret in the default namespace. This can take a few minutes**

**2. Install Integrify Cloud**

- Update the default values.yaml file downloaded from the Integrify helm charts repo with your installation information 

- Deploy release called integrify via helm. Make sure the `-f` flag is pointing to your values.yml file
```
helm install -f values.yaml integrify .
```

- It may take awhile to pull all the images and start the pods. If you get any sort of image pull error check the pod for the specific error. If it's a timeout error delete the pod and let it restart. 

- Once all pods are running you should be able to access your instance by opening a browser and going to http://integrify.demo


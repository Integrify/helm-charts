# Integrify-EKS
This README is an example walk-through of how Integrify creates a Kubernetes cluster in AWS EKS using eksctl and deploys the Integrify application using helm. This walk-through is to be used as an example only and you may need to change details for your cluster installation. **Deployment and support of Kubernetes is outside the scope of Integrify support.** 

# Setup An AWS EKS Cluster
For information and documentation on creating an AWS EKS cluster see Amazon's getting started guide - https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

**1. Deploy a new cluster using eksctl updating the name, region, vpc-cidr, AZs, and version as needed**
```
eksctl create cluster \
--name my-eks-cluster \
--region us-east-1 \
--vpc-cidr 192.168.110.0/24 \
--zones us-east-1a,us-east-1b \
--version 1.25 \
--with-oidc \
--fargate
```

- This should add a new context to your kubectl config. Check it by running
```
kubectl config get-contexts
```

- Swith to this context before deploying or updating the app
```
kubectl config use-context integrifyservice@internal-apps.us-east-1.eksctl.io
```

**2. Grant access to see cluster resources in AWS Console**
- https://docs.aws.amazon.com/eks/latest/userguide/view-kubernetes-resources.html#view-kubernetes-resources-permissions


 
**3. Add a namespace for Integrify Cloud**
```
kubectl create namespace my-namespace
```

**4. Add a fargate profile for the new namespace**
```
eksctl create fargateprofile \
--cluster my-eks-cluster \
--name my-namespace-fg-profile \
--namespace my-namespace \
--region us-east-1
```

**5. Download and create an IAM policy that will allow the load balancer controller to manage ELB resources**
```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
rm iam_policy.json
```

**6. Create an IAM role and serviceaccount for the load balancer controller**
- *Make sure to add the proper AWS Account ID*
- *This will be created by a CloudFormation stack*
```
eksctl create iamserviceaccount \
--cluster my-eks-cluster \
--region us-east-1 \
--namespace kube-system \
--name aws-load-balancer-controller \
--attach-policy-arn arn:aws:iam::{AWS_ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--approve
```

**7. Install the TargetGroupBinding CRDs**
```
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
```

**8. Install the load balanacer controller**

* Get your VPC ID
```
aws eks describe-cluster \
--name my-eks-cluster \
--region us-east-1 \
--query "cluster.resourcesVpcConfig.vpcId" \
--output text
```

* Add the helm repo for AWS
```
helm repo add eks https://aws.github.io/eks-charts
```
* Update the following with your VPC ID from above and run
```
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
--set clusterName=my-eks-cluster \
--set serviceAccount.create=false \
--set region=us-east-1 \
--set vpcId={VPC_ID} \
--set serviceAccount.name=aws-load-balancer-controller -n kube-system
```
* Verify the deployment
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```

**9. Install CloudWatch Logging** - *https://docs.aws.amazon.com/eks/latest/userguide/fargate-logging.html*

- Download yaml file from document above and create a custom logging namepsace
```
kubectl apply -f aws-observability-namespace.yaml
```

- Download the configmap yaml from above and apply the configmap

```
kubectl apply -f aws-logging-cloudwatch-configmap.yaml
```

- Download the CloudWatch IAM policy to your computer.
```
curl -o permissions.json https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/cloudwatchlogs/permissions.json
```

- Create an IAM policy from the policy file you downloaded in a previous step.
```
aws iam create-policy --policy-name eks-fargate-logging-policy --policy-document file://permissions.json
```
- Attach the IAM policy to the pod execution role specified for your Fargate profile with the following command. Replace 111122223333 with your account ID. Replace AmazonEKSFargatePodExecutionRole with your pod execution role (for more information

```
aws iam attach-role-policy \
--policy-arn arn:aws:iam::111122223333:policy/eks-fargate-logging-policy \
--role-name {amazon-fargate-eks-pod-execution-role}
```

**10. Create and setup Integrify databases**

  - ***You will need to contact [Integrify Support](https://support.integrify.com) and request the database schema scripts***
  - We recommend creating two databases for Integrify. A consumers database which will contain data about the Integrify installation and a data database which will be used to store all of the actual application data. There is no requirement for the database names, though we recommend naming them so you'll remember they are being used by Integrify. 
  - Create or add a user to the databases created above. Grant the user db_owner rights.
  - Run the scripts provided by Integrify Support to create the inital schemas
    - Run the *integrifyConsumers_FreshInstall_xx.xxxxx-U.sql* script against the consumers database to create the initial consumers schema
    - Run the *integrify_FreshInstall_xx.xxxxx-U.sql* script against the data database to create the initial data schema
    - *If you are only using one database, run both scripts against that one database*
  - Update the variables in the *integrifyConsumers_CreateTenant_xx.xxxxx.sql* file for your environment and then run that against the consumers database to add a new instance.

**11. Install Integrify**

- Download the default [values-eks.yaml](https://github.com/Integrify/helm-charts/blob/main/values-eks.yaml) template file from the Integrify Helm Charts GitHub page.

- Update the downloaded values-eks.yaml template file with your Integrify installation details and rename to values.yaml

- Add the Integrify Helm repo
```
helm repo add integrify https://integrify.github.io/helm-charts
```

- Deploy release called integrify via helm. Make sure the `-f` flag is pointing to your values.yml file
```
helm install -f values.yaml integrify integrify/integrify-eks
```

**12. Update DNS with load balancer DNS Name**

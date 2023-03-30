# Setup New Integrify Cloud Cluster

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

**10. Update the Integrify Cloud configmap and secrets file with installation specific information**

**11. Create the secrets and configmap in your custom namespace**
```
kubectl create secrets generic integrify-secrets \
--from-env-file secrets.env -n my-namespace
```

```
kubectl apply -n my-namespace -f configmap.yaml
```

**12. CD into helm-eks directory and deploy application via Helm**
```
helm install -n integrify integrify-cloud .
```

**13. Update DNS with load balancer DNS Name**

# Deploy an application in EKS and AWS Fargate

## 1 Setting up the AWS Environment for EKS
Creating an AWS account is the first step to access and utilize AWS services, including Amazon Elastic Kubernetes Service (EKS). Here's a step-by-step guide to creating an AWS account and setting up IAM users:
1. Create an AWS Account
2. Access AWS Management Console
3. Set up Multi-Factor Authentication (MFA) (Optional but recommended)
4. Create IAM Users
5. Access Keys (for Programmatic Access)

### 1.1 Configuring the AWS CLI, kubectl, eksctl

With IAM users set up, you can now configure the AWS CLI and kubectl on your local machine to interact with AWS services and EKS clusters:

1. **Installing the AWS CLI**:
   - Download and install the AWS CLI on your local machine. You can find installation instructions for various operating systems [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

2. **Configuring AWS CLI Credentials**:
   - Open a terminal or command prompt and run the following command:
     ```
     aws configure
     ```
   - Enter the access key ID and secret access key of the IAM user you created earlier.
   - Choose a default region and output format for AWS CLI commands.

3. **Installing kubectl**:
   - Install kubectl on your local machine. Instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

4. **Configuring kubectl for EKS**:
   - Once kubectl is installed, you need to configure it to work with your EKS cluster.
   - In the AWS Management Console, go to the EKS service and select your cluster.
   - Click on the "Config" button and follow the instructions to update your kubeconfig file. Alternatively, you can use the AWS CLI to update the kubeconfig file:
     ```
     aws eks update-kubeconfig --name your-cluster-name
     ```
   - Verify the configuration by running a kubectl command against your EKS cluster:
     ```
     kubectl get nodes
     ```
5. **Installing EKS**

   The AWS link to [install the AWS EKS](https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/setting-up-eksctl.html) on your machine. Aslo this the link to [eksctl tool](https://eksctl.io/installation/) webpage. 

   To creat the EKS cluster with fargate, run the following commands:
   - Install using Fargate
   ```
   eksctl create cluster --name demo-cluster --region us-east-1 --fargate
   ```
   - Delete the cluster
   ```
   eksctl delete cluster --name demo-cluster --region us-east-1
   ```

### 1.2 Preparing Networking and Security Groups for EKS

Before launching an EKS cluster, you need to prepare the networking and security groups to ensure proper communication and security within the cluster:

#### 1.2.1 **Creating an Amazon VPC (Virtual Private Cloud)**:
   - Go to the AWS Management Console and navigate to the VPC service.
   - Click on "Create VPC" and enter the necessary details like VPC name, IPv4 CIDR block, and subnets.
   - Create public and private subnets to distribute resources in different availability zones.

Sure! Let's go into detail for each of the points:

#### 1.2.2 **Configuring Security Groups**

**Security Groups** are a fundamental aspect of Amazon Web Services (AWS) that act as virtual firewalls for your AWS resources, including Amazon Elastic Kubernetes Service (EKS) clusters. Security Groups control inbound and outbound traffic to and from these resources based on rules you define. Here's a step-by-step guide on configuring Security Groups for your EKS cluster:

   1. **Create a Security Group**:
      - Go to the AWS Management Console and navigate to the Amazon VPC service.
      - Click on "Security Groups" in the left-hand navigation pane.
      - Click on "Create Security Group."
      - Provide a name and description for the Security Group.
      - Select the appropriate VPC for the Security Group.

   2. **Inbound Rules**:
      - Define inbound rules to control incoming traffic to your EKS worker nodes.
      - By default, all inbound traffic is denied unless you explicitly allow it.
      - Common inbound rules include allowing SSH (port 22) access for administrative purposes and allowing ingress traffic from specific CIDR blocks or Security Groups.

   3. **Outbound Rules**:
      - Define outbound rules to control the traffic leaving your EKS worker nodes.
      - By default, all outbound traffic is allowed unless you explicitly deny it.
      - For security purposes, you can restrict outbound traffic to specific destinations or ports.

   4. **Security Group IDs**:
      - After creating the Security Group, you'll receive a Security Group ID. This ID will be used when launching your EKS worker nodes.

   5. **Attach Security Group to EKS Worker Nodes**:
      - When launching the EKS worker nodes, specify the Security Group ID in the launch configuration. This associates the Security Group with the worker nodes, allowing them to communicate based on the defined rules.

Configuring Security Groups ensures that only the necessary traffic is allowed to and from your EKS worker nodes, enhancing the security of your EKS cluster.

### 1.3 **Setting Up Internet Gateway (IGW)**

An **Internet Gateway (IGW)** is a horizontally scaled, redundant, and highly available AWS resource that allows communication between your VPC and the internet. To enable EKS worker nodes to access the internet for tasks like pulling container images, you need to set up an Internet Gateway in your VPC. Here's how to do it:

1. **Create an Internet Gateway**:
   - Go to the AWS Management Console and navigate to the Amazon VPC service.
   - Click on "Internet Gateways" in the left-hand navigation pane.
   - Click on "Create Internet Gateway."
   - Provide a name for the Internet Gateway and click "Create Internet Gateway."

2. **Attach Internet Gateway to VPC**:
   - After creating the Internet Gateway, select the Internet Gateway in the list and click on "Attach to VPC."
   - Choose the VPC to which you want to attach the Internet Gateway and click "Attach."

3. **Update Route Tables**:
   - Go to "Route Tables" in the Amazon VPC service.
   - Identify the Route Table associated with the private subnets where your EKS worker nodes will be deployed.
   - Edit the Route Table and add a route with the destination `0.0.0.0/0` (all traffic) and the Internet Gateway ID as the target.

By setting up an Internet Gateway and updating the Route Tables, you provide internet access to your EKS worker nodes, enabling them to interact with external resources like container registries and external services.

### 1.4 **Configuring IAM Policies**

**Identity and Access Management (IAM)** is a service in AWS that allows you to manage access to AWS resources securely. IAM policies define permissions that specify what actions are allowed or denied on specific AWS resources. For your EKS cluster, you'll need to configure IAM policies to grant necessary permissions to your worker nodes and other resources. Here's how to do it:

1. **Create a Custom IAM Policy**:
   - Go to the AWS Management Console and navigate to the IAM service.
   - Click on "Policies" in the left-hand navigation pane.
   - Click on "Create policy."
   - Choose "JSON" as the policy language and define the permissions required for your EKS cluster. For example, you might need permissions for EC2 instances, Auto Scaling, Elastic Load Balancing, and accessing ECR (Elastic Container Registry).

2. **Attach the IAM Policy to IAM Roles**:
   - Go to "Roles" in the IAM service and select the IAM role that your EKS worker nodes will assume.
   - Click on "Attach policies" and search for the custom IAM policy you created in the previous step.
   - Attach the policy to the IAM role.

3. **Update EKS Worker Node Launch Configuration**:
   - When launching your EKS worker nodes, specify the IAM role ARN (Amazon Resource Name) of the IAM role that includes the necessary IAM policy.
   - The IAM role allows the worker nodes to authenticate with the EKS cluster and access AWS resources based on the permissions defined in the attached IAM policy.

By configuring IAM policies and associating them with IAM roles, you grant specific permissions to your EKS worker nodes, ensuring they can interact with AWS resources as needed while maintaining security and access control.

By completing these steps, your AWS environment is ready to host an Amazon EKS cluster. You can proceed with creating an EKS cluster using the AWS Management Console or AWS CLI as described in section 3.

## 2 The project commands
We are going to luanch a 2048 game in a k8s cluster. All the commands to run the project are
```
# 1. first configure the aws cli, then insert the required keys
# region : eu-central-1
$ aws configure 

# 2. To create the eks cluster
$ eksctl create cluster --name demo-cluster --region eu-central-1 --fargate

# 3. Change the kubectl context to the newly creately cluster in EKS
$ aws eks update-kubeconfig --region eu-central-1 --name demo-cluster

# 4. check the current k8s context
$ kubectx            # the kubectx tool should already been installed or 
$ kubectl config current-context

# 5. To create a new namespace for the app, first create the fargate profile
$ eksctl create fargateprofile \
    --cluster demo-cluster \
    --region eu-central-1 \
    --name alb-sample-app \
    --namespace game-2048

# 6. Now a new fargate profile with namespace of game-2048 is created in the compute tab of the cluster.

# 7. load all the k8s reources from the yaml file
$ kubectl apply -f https://raw.githubusercontent.com/daylook/sre/main/k8s/eks-fargate-game2048-resources.yml

# 8. To check if all the cluster resources are configured
$ kubectl get pods -n game-2048 -w
$ kubectl get svc -n game-2048
$ kubectl get ingress -n game-2048

# Note: We still dont have any address for the ingress, as the ingress controller still has not been installed

# 9. We need to add the oid provider to the cluster
$ eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve

# Now you can check the newly created oid provider in your IAM console

# We need to add the ALB ingress controller pod. (any controller in k8s is pod)
# Then we want to grant access to AWS service ALB to this pod.  
# This ALB ingress controller will create a load balancer for us.

# 10. First we need to create the IAM policy and role for it. 
# You can find the document in the alb controller documentations.

$ cd <root of the project>

$ aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://aws/eks-fargate-iam-policy.json

# Now you must be able to find the AWSLoadBalancerControllerIAMPolicy  in aws iam policies.

# 11. Create IAM Role
$ eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
  --override-existing-serviceaccounts

# Note: in case of error and for debugging purposes, if you want to delete the service accounts:
# A. go to the aws cloudFormation console 
# B. delete the stack which created this serviceAccount


# 12. Add helm repo
$ helm repo add eks https://aws.github.io/eks-charts
$ helm repo update eks

# 13. install alb from that helm-repo
$ helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>

# Note: the vpcId is in aws eks console -> networking tab of the cluser

# Note: in case of erro and need to delete the helm chart:
$ helm install aws-load-balancer-controller -n kube-system

# 14. Check if the load balancer has been created:
$ kubectl get deployments -n kube-system

# Note: to edit the deployment in case of error:
$ kubectl edit deploy/aws-load-balancer-controller -n kube-system
# then you can see the error in the 'status' part of the output.

# 15. Now we need to check if this ingress controller has created a ALB.
# so go to the 'aws ec2 alb' console, you should see that alb has been added.

# 16. Now we can check it with kubectl as well:
$ kubectl get ingress -n game-2048

# 17. you can extract the address from the output of the previous command, which is actually the DNS record of the load balancer. 
# Now you can run it in the browser with:
http://<ADDRESS> 

```

Note: As the ingress-controller is the cluster level appication which needed to be added to the cluster only once, you must first create the serviceAccount and attach that serviceAccount to the IAMRole for that ingress-controller.  



















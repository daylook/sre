# eks cluster practical example with fargate
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
$ kubectx            # the kubectx tool should already been installed 
or 
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



















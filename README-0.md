# EKS Cluster setup for Service Mesh

1. Create EKS cluster

# New VPC, private/public subnets, IGW will be created  
    a. 
    eksctl create cluster \
    --version 1.27 \
    --name cluster-2 \
    --region eu-central-1 \
    --nodegroup-name ng \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 3 \
    --nodes-max 5 \
    --vpc-cidr 10.0.0.0/16 \
    --vpc-nat-mode Disable

    b.
    export CLUSTER="cluster-2"
    export ACCOUNT_ID="590183829346"

2. Install CSI plugin

    https://catalog.workshops.aws/eks-immersionday/en-US/pesistentstorage-ebs/ebs-csi-driver

    a. Create OpenID Connect (OIDC) Provider

    eksctl utils associate-iam-oidc-provider \
    --region=eu-central-1 \
    --cluster=$CLUSTER \
    --approve


    b. Configure IAM Role for Service Account
        You can associate an IAM role with a Kubernetes service account. This service account can then provide AWS permissions to the containers in any pod that uses that service account

        aws iam create-policy \
    --policy-name AmazonEBSCSIDriverPolicy \
    --policy-document file://AmazonEBSCSIDriverPolicy.json

        (AmazonEBSCSIDriverPolicy: https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/example-iam-policy.json)


        # Create k8s service account and create IAM role and attached it with the policy
        eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster $CLUSTER \
        --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AmazonEBSCSIDriverPolicy \
        --approve \
        --role-only \
        --region=eu-central-1 \
        --role-name AmazonEKS_EBS_CSI_DriverRole

        eksctl get iamserviceaccount --cluster=${CLUSTER} --region=eu-central-1
        k get sa -n kube-system | grep ebs-csi


    c. Add EBS CSI add-on

    eksctl create addon --name aws-ebs-csi-driver --cluster $CLUSTER \
        --service-account-role-arn arn:aws:iam::${ACCOUNT_ID}:role/AmazonEKS_EBS_CSI_DriverRole --region=eu-central-1 --force

    k get pods -n kube-system | grep ebs-csi
    eksctl get addon --name aws-ebs-csi-driver --cluster $CLUSTER --region=eu-central-1 


    d. Define Storage Class
    Dynamic Volume Provisioning  allows storage volumes to be created on-demand

    k create -f mysql-storageclass.yaml


3. Install ALB controller

    a.
    eksctl utils associate-iam-oidc-provider \
    --region=eu-central-1 \
    --cluster=$CLUSTER \
    --approve

    b.
    aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://AWSLoadBalancerControllerIAMPolicy.json

    eksctl create iamserviceaccount \
    --cluster=${CLUSTER} \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --region=eu-central-1 \
    --approve

    k get sa -n kube-system | grep aws-load-balancer

    c. 
    helm repo add eks https://aws.github.io/eks-charts && helm repo update eks

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=${CLUSTER} --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

    k get pods -n kube-system | grep aws-load-balancer
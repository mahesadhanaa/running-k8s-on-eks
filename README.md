# running-k8s-on-eks
how to running kubernetes on EKS AWS

Requirements for EKS deployments :


1. Create IAM roles and policies

    In order to access the EKS cluster services (CRUD opreations, and even just being able to connect to the cluster), an IAM policy is needed. The following, while "open" only allows eks service access, and is appropriate for most users who need to manipulate the EKS service

    - Create a policy with the following json object, and name it xxxxx

      iam_eks_policy.json

            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": [
                            "eks:*"
                        ],
                        "Resource": "*"
                    }
                ]
            }

    - and also need CloudFormation access policy which we can call xxxx

      iam_cf_policy.json

            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": "*",
                        "Resource": "*"
                    }
                ]
            }
    - Create Role for EKS (role ARN)           

2. Create admin and kubernetes users

   While we can add policy to our normal users in AWS, for this class we will create two users, an admin user and a second eks system user.

   User 1:

   clusterAdmin
   - eks admin policy
   - k8s admin "system:master" group
     - the following policies:
       - AdminEKSPolicy
       - AdminCloudFormationPolicy
       - AmazonEKSServicePolicy
       - AmazonEKSClusterPolicy

   User 2:

   clusterUser
   - no IAM policies
   - k8s admin "system:master" group
     - the following policies:
       - AdminEKSPolicy

   for both users, create programatic credentials, and for the admin user, create a password credential as well.

3. Build the EKS VPC

   - Create VPC, Subnet, Internet Gateway, Security Group
   - Or you can use cloudformation template from S3 template : https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-09/amazon-eks-vpc-sample.yaml

4. Install kubectl for EKS auth
   
   - Install kubectl as normal from the instructions found here:
     
     https://kubernetes.io/docs/tasks/tools/install-kubectl/

   -  then also need the aws-iam-authenticator binary:
     
     https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html

   -  chmod the both, and move to directory /usr/local/bin/

   - Install the AWS CLI :
     
     https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html


5. Launch an EKS cluster master

   Once we have created our IAM roles policies, and generated our VPC we've ready to launch our EKS control plane.

   we can launch via the EKS console.

   when have done. the first we need to log into the CLI using the key and secret we downloaded for the clusterAdmin :

   $ aws configure --profile=clusterAdmin

     AWS Access Key ID --> Access Key clusterAdmin

     AWS Secret Access Key -->  Secret Access Key clusterAdmin

     Default region name --> region name

     Default output format 

   then, we update kubeconfig for remote access EKS :

   $ aws eks update-kubeconfig --name clustername

   And, lastly, we should be able to confirm our access

   $ kubectl get nodes

   $ kubectl get pods

6. Selecting worker sizing

   - you use cloudformation for create workers. and then S3 template for create eks worker : https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-01-09/amazon-eks-nodegroup.yaml

   - After complete, we can go ahead and grab our instance role

   - and then we can create aws auth configmap yaml:

    [

        apiVersion: v1
        kind: ConfigMap
        metadata:
        name: aws-auth
        namespace: kube-system
        data:
        mapRoles: |
            - rolearn: 
            username: system:node:{{EC2PrivateDNSName}}
            groups:
                - system:bootstrappers
                - system:nodes

    ]

    - now, apply the yaml configmap 

      $ kubectl apply -f configmapfilename.yaml

      check nodes worker

      $ kubectl get nodes

    the worker is ready !!!
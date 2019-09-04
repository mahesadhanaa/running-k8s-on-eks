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
     the following policies:
     AdminEKSPolicy
     AdminCloudFormationPolicy
     AmazonEKSServicePolicy
     AmazonEKSClusterPolicy

   User 2:

   clusterUser
   - no IAM policies
   - k8s admin "system:master" group
     the following policies:
     AdminEKSPolicy

   for both users, create programatic credentials, and for the admin user, create a password credential as well.

3. Build the EKS VPC

4. Install kubectl for EKS auth

5. Launch an EKS cluster master

6. Selecting worker sizing

7. Create a worker scale policy
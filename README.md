# Karpenter
A collection of resources detailing the Karpenter installation process and offering resolutions for potential errors faced both during setup and ongoing operation.

# Before you can start the installation process you must complete these prerequsites. 
1. Create OIDC provider for your EKS cluster.
2. Create Karpenter node IAM role with below permissions and trust relationship policy.

Trust relationship.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
**Permissions:**

Attach below AWS managed permissions to your karppenter IAM role.
- AmazonEKSWorkerNodePolicy
- AmazonEC2ContainerRegistryPullOnly 
- AmazonEKS_CNI_Policy
- AmazonSSMManagedInstanceCore

3. Create a karpenter controller IAM role with below trust relationship and json policy.

Trust relationship
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:oidc-provider/${OIDC_ENDPOINT#*//}"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "${OIDC_ENDPOINT#*//}:aud": "sts.amazonaws.com",
                    "${OIDC_ENDPOINT#*//}:sub": "system:serviceaccount:karpenter:karpenter"
                }
            }
        }
    ]
}
```

Json policy

```
{
    "Statement": [
        {
            "Action": [
                "ssm:GetParameter",
                "ec2:DescribeImages",
                "ec2:RunInstances",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeLaunchTemplates",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceTypes",
                "ec2:DescribeInstanceTypeOfferings",
                "ec2:DescribeAvailabilityZones",
                "ec2:DeleteLaunchTemplate",
                "ec2:CreateTags",
                "ec2:CreateLaunchTemplate",
                "ec2:CreateFleet",
                "ec2:DescribeSpotPriceHistory",
                "pricing:GetProducts"
            ],
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "Karpenter"
        },
        {
            "Action": "ec2:TerminateInstances",
            "Condition": {
                "StringLike": {
                    "ec2:ResourceTag/karpenter.sh/provisioner-name": "*"
                }
            },
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "ConditionalEC2Termination"
        },
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:role/KarpenterNodeRole-${CLUSTER_NAME}",
            "Sid": "PassNodeIAMRole"
        },
        {
            "Effect": "Allow",
            "Action": "eks:DescribeCluster",
            "Resource": "arn:${AWS_PARTITION}:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${CLUSTER_NAME}",
            "Sid": "EKSClusterEndpointLookup"
        }
    ],
    "Version": "2012-10-17"
}
```
While creating a roles and policy make sure you used correcr 'AWS_ACCOUNT_ID', 'AWS_REGION', 'CLUSTER_NAME' and 'AWS_PARTITION'.

To dive deep further, you can use below documents as referance:
[1] https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/
[2] https://www.eksworkshop.com/docs/autoscaling/compute/karpenter/configure

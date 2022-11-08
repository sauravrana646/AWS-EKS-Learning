First if it is a private cluster then we need to create a EC2 instace which will act as bastion host and have admin access to cluster. 

All the commands for managing and deploying to Kubernetes will be run from this intance. So for that we need to follow below steps:
- Launch an EC2 instace in the same VPC as the EKS or allow public endpoint access for API and whitelist IP of EC2.
- Create a AWS IAM Policy with EKS Read Access.
- Create a role with above policy attached with EC2 as trusted entity in trust relationship for sts assume role action.
- Now attach the role with EC2 instance launched with bastion host.
- Also important step is to allow this instance to access API endpoint of EKS. This is done by allowing EC2 instance IP or security group in the EKS Control Plane Security Group.
- Add this role arn in the Identity Mappings section of the `eksctl` config YAML file.
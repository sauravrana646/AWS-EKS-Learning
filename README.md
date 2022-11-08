# AWS EKS

#### Important Links

[Managed node group errors]([https://](https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html#troubleshoot-managed-node-groups)

[Private Cluster Considerations]([https://](https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html)

[Enabling IAM user and role access to your cluster]([https://](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)

[Troubleshooting]([https://](https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html)

[Bootstrap with containerd instead of docker](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html#containerd-bootstrap)

[EKS Optimized Amazon AMI](https://docs.aws.amazon.com/eks/latest/userguide/eks-optimized-ami.html)

[EKS ECR Issues]([https://](https://aws.amazon.com/premiumsupport/knowledge-center/eks-ecr-troubleshooting/)



#### Two options to create a cluster: 

- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
- [AWS console or AWS CLI ](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)

### Important Points

- Managed node groups can't be deployed on AWS Outposts or in AWS Wavelength or AWS Local Zones.


### Things to keep in mind:

#### Stateful Applications
If you are running a stateful application across multiple Availability Zones that is backed by Amazon EBS volumes and using the Kubernetes [Cluster Autoscaler](https://docs.aws.amazon.com/eks/latest/userguide/autoscaling.html#cluster-autoscaler), you should configure multiple node groups, each scoped to a single Availability Zone. In addition, you should enable the --balance-similar-node-groups feature.

#### **IAM Permissions**
 The IAM entity (user or role) that created the cluster is the only IAM entity that can make calls to the Kubernetes API server with kubectl or the AWS Management Console. If you want other IAM users or roles to have access to your cluster, then you need to add them.

#### **VPC requirements and considerations** 
 When you create a cluster, you specify a VPC and at least two subnets that are in different Availability Zones. This topic provides an overview of Amazon EKS specific requirements and considerations for the VPC and subnets that you use with your cluster. If you don't have a VPC to use with Amazon EKS, you can create one using an Amazon EKS provided AWS CloudFormation template.

 When you create a cluster, the VPC that you specify must meet the following requirements and considerations:
 - The VPC must have a sufficient number of IP addresses available for the cluster, any nodes, and other Kubernetes resources that you want to create. If the VPC that you want to use doesn't have a sufficient number of IP addresses, try to increase the number of available IP addresses. You can do this by associating additional Classless Inter-Domain Routing (CIDR) blocks with your VPC. You can associate private (RFC 1918) and public (non-RFC 1918) CIDR blocks to your VPC either before or after you create your cluster. It can take a cluster up to five hours for a CIDR block that you associated with a VPC to be recognized.
 You can conserve IP address utilization by using a transit gateway with a shared services VPC.
 - **The VPC must have DNS hostname and DNS resolution support. Otherwise, nodes can't register to your cluster.**

#### **Subnet requirements and considerations** 
 When you create a cluster, Amazon EKS creates 2–4 elastic network interfaces in the subnets that you specify. These network interfaces enable communication between your cluster and your VPC. These network interfaces also enable Kubernetes features such as kubectl exec and kubectl logs. Each Amazon EKS created network interface has the text Amazon EKS cluster-name in its description.

 Amazon EKS can create its network interfaces in any subnet that you specify when you create a cluster. You can't change which subnets Amazon EKS creates its network interfaces in after your cluster is created.

 When you update the Kubernetes version of a cluster, Amazon EKS deletes the original network interfaces that it created, and creates new network interfaces. These network interfaces might be created in the same subnets as the original network interfaces or in different subnets than the original network interfaces. To control which subnets network interfaces are created in, you can limit the number of subnets you specify to only two when you create a cluster. 

 The subnets that you specify when you create a cluster must meet the following requirements:

 - The subnets must each have at least six IP addresses for use by Amazon EKS. However, we recommend at least 16 IP addresses.
 - The subnets must use IP address based naming. Amazon EC2 resource-based naming isn't supported with Amazon EKS.
 - The subnets can be a public or private. However, we recommend that you specify private subnets, if possible. A public subnet is a subnet with a route table that includes a route to an internet gateway, whereas a private subnet is a subnet with a route table that doesn't include a route to an internet gateway.
 
 You can deploy nodes and Kubernetes resources to the same subnets that you specify when you create your cluster. However, this isn't necessary. This is because you can also deploy nodes and Kubernetes resources to subnets that you didn't specify when you created the cluster. If you deploy nodes to different subnets, Amazon EKS doesn't create cluster network interfaces in those subnets. Any subnet that you deploy nodes and Kubernetes resources to must meet the following requirements:
 - The subnets must have enough available IP addresses to deploy all of your nodes and Kubernetes resources to.
 - The subnets must use IP address-based naming. Amazon EC2 resource-based naming isn't supported with Amazon EKS.
 - **If you need inbound access from the internet to your pods, make sure to have at least one public subnet with enough available IP addresses to deploy load balancers and ingresses to. You can deploy load balancers to public subnets. Load balancers can load balance to pods in private or public subnets. We recommend deploying your nodes to private subnets, if possible.**
 - If you plan to deploy nodes to a public subnet, the subnet must auto-assign IPv4 public addresses or IPv6 addresses. If you deploy nodes to a private subnet that has an associated IPv6 CIDR block, the private subnet must also auto-assign IPv6 addresses. If you used an Amazon EKS AWS CloudFormation template to deploy your VPC after March 26, 2020, this setting is enabled. If you used the templates to deploy your VPC before this date or you use your own VPC, you must enable this setting manually.
 - If the subnet that you deploy a node to is a private subnet and its route table doesn't include a route to a network address translation (NAT) device (IPv4) or an egress-only gateway (IPv6), add VPC endpoints using AWS PrivateLink to your VPC. VPC endpoints are needed for all the AWS services that your nodes and pods need to communicate with. Examples include Amazon ECR, Elastic Load Balancing, Amazon CloudWatch, AWS Security Token Service, and Amazon Simple Storage Service (Amazon S3). The endpoint must include the subnet that the nodes are in. Not all AWS services support VPC endpoints.
 - **If you want to deploy load balancers to a subnet, the subnet must have the following tag:**
    - Private subnets: 
        | Key                               | Value |
        | --------------------------------- | ----- |
        | `kubernetes.io/role/internal-elb` | 1     |
    - Public subnets:
        | Key                      | Value |
        | ------------------------ | ----- |
        | `kubernetes.io/role/elb` | 1     |

 -  If the node group is deployed to a public subnet on or after April 22, 2020, automatic assignment of public IP addresses must be enabled for the public subnet.

#### **EKS security group requirements and considerations**
   - When you create a cluster, Amazon EKS creates a security group that's named `eks-cluster-sg-my-cluster-uniqueID`. This security group has the following default rules:
        | Rule type | Protocol | Ports | Source | Destination                     |
        | --------- | -------- | ----- | ------ | ------------------------------- |
        | Inbound   | All      | All   | Self   |                                 |
        | Outbound  | All      | All   |        | 0.0.0.0/0 (IPv4) or ::/0 (IPv6) |

        Amazon EKS tags this security group with the following tags:

        | Key                                | Value      |
        | ---------------------------------- | ---------- |
        | `kubernetes.io/cluster/my-cluster` | owned      |
        | `aws:eks:cluster-name`             | my-cluster |

  - Amazon EKS automatically associates this security group to the following resources that it also creates:

      - 2–4 elastic network interfaces (referred to for the rest of this document as network interface) that are created when you create your cluster.

      - Network interfaces of the nodes in any managed node group that you create.

    - The default rules allow all traffic to flow freely between your cluster and nodes, and allows all outbound traffic to any destination. When you create a cluster, you can (optionally) specify your own security groups. If you do, then **Amazon EKS also associates the security groups that you specify to the network interfaces that it creates for your cluster. However, it doesn't associate them to any node groups that you create.**

#### Managed Node Group Considerations 
 - Amazon EKS managed node groups automate the provisioning and lifecycle management of nodes (Amazon EC2 instances) for Amazon EKS Kubernetes clusters.
 With Amazon EKS managed node groups, you don’t need to separately provision or register the Amazon EC2 instances that provide compute capacity to run your Kubernetes applications. You can create, automatically update, or terminate nodes for your cluster with a single operation.
 - **Node updates and terminations automatically drain nodes to ensure that your applications stay available.**
 - Every managed node is provisioned as part of an Amazon EC2 Auto Scaling group that's managed for you by Amazon EKS. Every resource including the instances and Auto Scaling groups runs within your AWS account. Each node group runs across multiple Availability Zones that you define.
 - Nodes launched as part of a managed node group are automatically tagged for auto-discovery by the Kubernetes cluster autoscaler. You can use the node group to apply Kubernetes labels to nodes and update them at any time
    
    ##### Managed node groups concepts
    
    - Amazon EKS managed node groups create and manage Amazon EC2 instances for you.

    - Every managed node is provisioned as part of an Amzon EC2 Auto Scaling group that's managed for you by Amazon EKS. Moreover, every resource including Amazon EC2 instances and Auto Scaling groups run within your AWS account.

    - **The Auto Scaling group of a managed node group spans every subnet that you specify when you create the group.**

    - **Amazon EKS tags managed node group resources so that they are configured to use the Kubernetes Cluster Autoscaler.**
    </br>
    > NOTE: If you are running a stateful application across multiple Availability Zones that is backed by Amazon EBS volumes and using the Kubernetes Cluster Autoscaler, you should configure multiple node groups, each scoped to a single Availability Zone. In addition, you should enable the --balance-similar-node-groups feature.
    - Amazon EKS managed node groups can be launched in both public and private subnets. If you launch a managed node group in a public subnet on or after April 22, 2020, the subnet must have MapPublicIpOnLaunch set to true for the instances to successfully join a cluster. If the public subnet was created using eksctl or the Amazon EKS vended AWS CloudFormation templates on or after March 26, 2020, then this setting is already set to true.
    - When using VPC endpoints in private subnets, you must create endpoints for `com.amazonaws.region.ecr.api`, `com.amazonaws.region.ecr.dkr`, and a `gateway endpoint for Amazon S3`.
    - **Managed node groups can't be deployed on AWS Outposts or in AWS Wavelength or AWS Local Zones.**
    - **Amazon EKS adds Kubernetes labels to managed node group instances. These Amazon EKS provided labels are prefixed with eks.amazonaws.com.**
    - Amazon EKS automatically drains nodes using the Kubernetes API during terminations or updates. Updates respect the pod disruption budgets that you set for your pods.
    - If you want to **encrypt Amazon EBS volumes** for your nodes, you can deploy the nodes using a launch template. To deploy managed nodes with encrypted Amazon EBS volumes without using a launch template, encrypt all new Amazon EBS volumes created in your account. For more information, see Encryption by default in the Amazon EC2 User Guide for Linux Instances.




### Pre-requisites

#### **Required IAM permissions** 
 The IAM security principal that you're using must have permissions to work with Amazon EKS IAM roles and service linked roles, AWS CloudFormation, and a VPC and related resources
#### **Creating a VPC for EKS** 
  - **Public and Private Subnets**
    - This VPC has two public and two private subnets. A public subnet's associated route table has a route to an internet gateway. However, the route table of a private subnet doesn't have a route to an internet gateway. One public and one private subnet are deployed to the same Availability Zone. The other public and private subnets are deployed to a second Availability Zone in the same AWS Region. We recommend this option for most deployments.

    - With this option, you can deploy your nodes to private subnets. This option allows Kubernetes to deploy load balancers to the public subnets that can load balance traffic to pods that run on nodes in the private subnets. Public IPv4 addresses are automatically assigned to nodes that are deployed to public subnets, but public IPv4 addresses aren't assigned to nodes deployed to private subnets.

    -  The nodes in private subnets can communicate with the cluster and other AWS services. Pods can communicate to the internet through a NAT gateway using IPv4 addresses or outbound-only Internet gateway using IPv6 addresses deployed in each Availability Zone. A security group is deployed that has rules that deny all inbound traffic from sources other than the cluster or nodes but allows all outbound traffic. The subnets are tagged so that Kubernetes can deploy load balancers to them.
    -  [Creating a VPC]([https://](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html))
   - **Only Public Subnets**
     - This VPC has three public subnets that are deployed into different Availability Zones in an AWS Region. All nodes are automatically assigned public IPv4 addresses and can send and receive internet traffic through an internet gateway. A security group is deployed that denies all inbound traffic and allows all outbound traffic. The subnets are tagged so that Kubernetes can deploy load balancers to them.
   - **Only Private Subnets**
     - This VPC has three private subnets that are deployed into different Availability Zones in the AWS Region. Resources that are deployed to the subnets can't access the internet, nor can the internet access resources in the subnets. The template creates VPC endpoints using AWS PrivateLink for several AWS services that nodes typically need to access. If your nodes need outbound internet access, you can add a public NAT gateway in the Availability Zone of each subnet after the VPC is created. A security group is created that denies all inbound traffic, except from resources deployed into the subnets. A security group also allows all outbound traffic. The subnets are tagged so that Kubernetes can deploy internal load balancers to them. If you're creating a VPC with this configuration, see [Private cluster requirements](https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html) for additional requirements and considerations.



### Method 1: Using AWS Console

- #### Step1: Create your Amazon EKS cluster

  1. Create an Amazon VPC with public and private subnets that meets Amazon EKS requirements.(See Pre-requisites Section)
   </br>
  >NOTE: _Some AZs and Regions are not supported by EKS_

  1. To creat the VPC we can either do it manually or we can use cloudformation stack to create a VPC with recommended settings for us. 

  ```
  aws cloudformation create-stack \
    --region region-code \
    --stack-name my-eks-vpc-stack \
    --template-url https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
  ```

  3. Create a cluster IAM role and attach the required Amazon EKS IAM managed policy to it. Kubernetes clusters managed by Amazon EKS make calls to other AWS services on your behalf to manage the resources that you use with the service.
      - Copy the following contents to a file named eks-cluster-role-trust-policy.json.
          ```
              {
          "Version": "2012-10-17",
          "Statement": [
              {
              "Effect": "Allow",
              "Principal": {
                  "Service": "eks.amazonaws.com"
              },
              "Action": "sts:AssumeRole"
              }
          ]
          }
          ```
      - Create the role.
          ```
          aws iam create-role \
          --role-name myAmazonEKSClusterRole \
          --assume-role-policy-document file://"eks-cluster-role-trust-policy.json"
          ```
      - Attach the required Amazon EKS managed IAM policy to the role.
          ```
          aws iam attach-role-policy \
          --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy \
          --role-name myAmazonEKSClusterRole
          ```
      - Choose Add cluster, and then choose Create. If you don't see this option, then choose Clusters in the left navigation pane first.
      - On the Configure cluster page, do the following:
        - Enter a Name for your cluster, such as my-cluster. The name can contain only alphanumeric characters (case-sensitive) and hyphens. It must start with an alphabetic character and can't be longer than 100 characters.
        - For Cluster Service Role, choose myAmazonEKSClusterRole.
        - Leave the remaining settings at their default values and choose Next.
      - On the Specify networking page, do the following:
        - Choose the ID of the VPC that you created in a previous step from the VPC dropdown list. It is something like vpc-00x0000x000x0x000 | my-eks-vpc-stack-VPC.
        - Leave the remaining settings at their default values and choose Next.
      - On the Configure logging page, choose Next.
      - On the Review and create page, choose Create.
    
    > NOTE: You might receive an error that one of the Availability Zones in your request doesn't have sufficient capacity to create an Amazon EKS cluster. If this happens, the error output contains the Availability Zones that can support a new cluster. Retry creating your cluster with at least two subnets that are located in the supported Availability Zones for your account
- #### Step 2: Configure your computer to communicate with your cluster 
 In this section, you create a kubeconfig file for your cluster. The settings in this file enable the kubectl CLI to communicate with your cluster.
 To configure your computer to communicate with your cluster
  1. Create or update a kubeconfig file for your cluster. Replace region-code with the AWS Region that you created your cluster in. Replace my-cluster with the name of your cluster.
   `aws eks update-kubeconfig --region region-code --name my-cluster`
  2. Test your configuration.
- #### Step 3: Create nodes
  - **Managed Nodes Linux**:
   To create your Amazon EC2 Linux managed node group:
    1. Create a node IAM role and attach the required Amazon EKS IAM managed policy to it. The Amazon EKS node kubelet daemon makes calls to AWS APIs on your behalf. Nodes receive permissions for these API calls through an IAM instance profile and associated policies.
        a. Copy the following contents to a file named node-role-trust-policy.json.
          ```
          {
            "Version": "2012-10-17",
            "Statement": [
              {
                "Effect": "Allow",
                "Principal": {
                  "Service": "ec2.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
              }
            ]
          }
          ```
        b. Create the node IAM role.
            ```
            aws iam create-role \
            --role-name myAmazonEKSNodeRole \
            --assume-role-policy-document file://"node-role-trust-policy.json"
            ```
        c. Attach the required managed IAM policies to the role.

              ```
              aws iam attach-role-policy \
              --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy \
              --role-name myAmazonEKSNodeRole
            aws iam attach-role-policy \
              --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
              --role-name myAmazonEKSNodeRole
            aws iam attach-role-policy \
              --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
              --role-name myAmazonEKSNodeRole
              ```
    2. Then create Node Group from the EKS Section with the Role configured above.

### Method 2: eksctl     

### Amazon EKS cluster endpoint access control 
  This topic helps you to enable private access for your Amazon EKS cluster's Kubernetes API server endpoint and limit, or completely disable, public access from the internet.

  When you create a new cluster, Amazon EKS creates an endpoint for the managed Kubernetes API server that you use to communicate with your cluster (using Kubernetes management tools such as kubectl). By default, this API server endpoint is public to the internet, and access to the API server is secured using a combination of AWS Identity and Access Management (IAM) and native Kubernetes Role Based Access Control (RBAC).

  You can enable private access to the Kubernetes API server so that all communication between your nodes and the API server stays within your VPC. You can limit the IP addresses that can access your API server from the internet, or completely disable internet access to the API server.

  **When you enable endpoint private access for your cluster, Amazon EKS creates a Route 53 private hosted zone on your behalf and associates it with your cluster's VPC. This private hosted zone is managed by Amazon EKS, and it doesn't appear in your account's Route 53 resources. In order for the private hosted zone to properly route traffic to your API server, your VPC must have enableDnsHostnames and enableDnsSupport set to true, and the DHCP options set for your VPC must include AmazonProvidedDNS in its domain name servers list. For more information, see Updating DNS support for your VPC in the Amazon VPC User Guide.**
  
  You can define your API server endpoint access requirements when you create a new cluster, and you can update the API server endpoint access for a cluster at any time. 

  | Endpoint public access | Endpoint private access | Behavior                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
  | ---------------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | Enabled                | Disabled                | This is the default behavior for new Amazon EKS clusters.</br></br>Kubernetes API requests that originate from within your cluster's VPC (such as node to control plane communication) leave the VPC but not Amazon's network.</br></br>Your cluster API server is accessible from the internet. You can, optionally, limit the CIDR blocks that can access the public endpoint. If you limit access to specific CIDR blocks, then it is recommended that you also enable the private endpoint, or ensure that the CIDR blocks that you specify include the addresses that nodes and Fargate pods (if you use them) access the public endpoint from.                                                                                                                                                                                                                                                                        |
  | Enabled                | Disabled                | Kubernetes API requests within your cluster's VPC (such as node to control plane communication) use the private VPC endpoint.</br></br>Your cluster API server is accessible from the internet. You can, optionally, limit the CIDR blocks that can access the public endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
  | Enabled                | Disabled                | All traffic to your cluster API server must come from within your cluster's VPC or a connected network.</br></br>There is no public access to your API server from the internet. Any kubectl commands must come from within the VPC or a connected network. For connectivity options, see Accessing a [private only API server.](https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html#private-access) </br></br>The cluster's API server endpoint is resolved by public DNS servers to a private IP address from the VPC. In the past, the endpoint could only be resolved from within the VPC.</br></br>**If your endpoint does not resolve to a private IP address within the VPC for an existing cluster, you can:**</br></br>Enable public access and then disable it again. You only need to do so once for a cluster and the endpoint will resolve to a private IP address from that point forward. |


### Private cluster requirements

This topic describes how to deploy an Amazon EKS cluster that doesn't have outbound internet access. If you're not familiar with Amazon EKS networking, see [De-mystifying cluster networking for Amazon EKS worker nodes](http://aws.amazon.com/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/). If your cluster doesn't have outbound internet access, then it must meet the following requirements:
  - Your cluster must pull images from a container registry that's in your VPC. You can create an Amazon Elastic Container Registry in your VPC and copy container images to it for your nodes to pull from.
  - Your cluster must have endpoint private access enabled. This is required for nodes to register with the cluster endpoint. Endpoint public access is optional. For more information, see Amazon EKS cluster endpoint access control.
  - Self-managed Linux and Windows nodes must include the following bootstrap arguments before they're launched. These arguments bypass Amazon EKS introspection and don't require access to the Amazon EKS API from within the VPC.
    - Determine the value of your cluster's endpoint with the following command. Replace my-cluster with the name of your cluster.
        ```
        aws eks describe-cluster --name my-cluster --query cluster.endpoint --output text
        ```
        The example output is as follows.
        ```
        https://EXAMPLE108C897D9B2F1B21D5EXAMPLE.sk1.region-code.eks.amazonaws.com
        ```
    - Determine the value of your cluster's certificate authority with the following command. Replace my-cluster with the name of your cluster.
        ```
        aws eks describe-cluster --name my-cluster --query cluster.certificateAuthority --output text
        ```
        The returned output is a long string.
    - Replace cluster-endpoint and certificate-authority in the following commands with the values returned in the output from the previous commands. For more information about specifying bootstrap arguments when launching self-managed nodes, see Launching self-managed Amazon Linux nodes and Launching self-managed Windows nodes.
      - For Linux nodes:
          ```
          --apiserver-endpoint cluster-endpoint --b64-cluster-ca certificate-authority
          ```
          For additional arguments, see the [bootstrap script](https://github.com/awslabs/amazon-eks-ami/blob/master/files/bootstrap.sh) on GitHub.
          </br>
  - **Your cluster's aws-auth ConfigMap must be created from within your VPC. For more information about create the aws-auth ConfigMap, see Enabling IAM user and role access to your cluster.**
  - Pods configured with IAM roles for service accounts acquire credentials from an AWS Security Token Service (AWS STS) API call. **If there is no outbound internet access, you must create and use an AWS STS VPC endpoint in your VPC**. For more information, see [Configuring the AWS Security Token Service endpoint for a service account](https://docs.aws.amazon.com/eks/latest/userguide/configure-sts-endpoint.html).
  - Your cluster's VPC subnets must have a VPC interface endpoint for any AWS services that your pods need access to. For more information, see [Access an AWS service using an interface VPC endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html). Some commonly-used services and endpoints are listed in the following table. For a complete list of endpoints, see AWS services that integrate with AWS PrivateLink in the AWS PrivateLink Guide.
    | Service                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Endpoint                                                                                               |
    | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
    | Amazon EC2                                                                                                                                                                                                                                                                                                                                                                                                                                                          | com.amazonaws.region-code.ec2                                                                          |
    | Amazon Elastic Container Registry (for pulling container images)                                                                                                                                                                                                                                                                                                                                                                                                    | com.amazonaws.region-code.ecr.api, com.amazonaws.region-code.ecr.dkr, and com.amazonaws.region-code.s3 |
    | Application Load Balancers and Network Load Balancers                                                                                                                                                                                                                                                                                                                                                                                                               | com.amazonaws.region-code.elasticloadbalancing                                                         |
    | Amazon CloudWatch Logs                                                                                                                                                                                                                                                                                                                                                                                                                                              | com.amazonaws.region-code.logs                                                                         |
    | AWS Security Token Service (required when using IAM roles for service accounts)                                                                                                                                                                                                                                                                                                                                                                                     | com.amazonaws.region-code.sts                                                                          |
    | App Mesh</br></br>1. The App Mesh controller for Kubernetes isn't supported. For more information, see App Mesh controller on GitHub.</br></br>2. Cluster Autoscaler is supported. When deploying Cluster Autoscaler pods, make sure that the command line includes --aws-use-static-instance-list=true. For more information, see Use Static Instance List on GitHub. The worker node VPC must also include the AWS STS VPC endpoint and autoscaling VPC endpoint. | com.amazonaws.region-code.appmesh-envoy-management                                                     |
    ##### Considerations
    - Any self-managed nodes must be deployed to subnets that have the VPC interface endpoints that you require. If you create a managed node group, the VPC interface endpoint security group must allow the CIDR for the subnets, or you must add the created node security group to the VPC interface endpoint security group.
    - If your pods use Amazon EFS volumes, then before deploying the [Amazon EFS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html), the driver's [kustomization.yaml](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/deploy/kubernetes/overlays/stable/kustomization.yaml) file must be changed to set the container images to use the same AWS Region as the Amazon EKS cluster.
    - You can use the [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) to deploy AWS Application Load Balancers (ALB) and Network Load Balancers to your private cluster. When deploying it, you should use command line flags to set enable-shield, enable-waf, and enable-wafv2 to false.
    - **Certificate discovery with hostnames from Ingress objects isn't supported. This is because the controller needs to reach AWS Certificate Manager, which doesn't have a VPC interface endpoint.**

# Deploying Netskope Cloud Exchange using AWS ECS Fargate
<div style="text-align: justify">
The Netskope Cloud Exchange (CE) provides customers with powerful integration tools to leverage investments across their security posture.

Cloud Exchange consumes valuable Netskope telemetry and external threat intelligence and risk scores, enabling improved policy implementation, automated service ticket creation, and exportation of log events from the Netskope Security Cloud. 

To learn more about Netskope Cloud Exchange please refer to the [Netskope Cloud Exchange introduction page](https://www.netskope.com/products/capabilities/cloud-exchange).

Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that makes it easy for you to deploy, manage and scale containerized applications.

AWS Fargate is a serverless, pay-as-you-go compute engine that lets you focus on building applications without managing servers. AWS Fargate is compatible with Amazon Elastic Container Service (ECS). To learn more about Amazon ECS please follow the [Amazon ECS](https://aws.amazon.com/ecs/) documentation page. To learn more about AWS Fargate please follow the [AWS Fargate](https://aws.amazon.com/fargate/) documentation page.

AWS CloudFormation is a service that helps you model and set up your AWS resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. To learn more about AWS CloudFormation please follow the [AWS CloudFormation](https://aws.amazon.com/cloudformation/) documentation page.

This document will guide you on how to deploy Netskope Cloud Exchange on AWS Fargate using AWS CloudFormation. </div>

This solution consists of the following components:

**CloudExchangeTemplate.yaml** - AWS CloudFormation template that deploys the following resources:

- Networking Resources - VPC, Private Subnets (ECS Cluster, ALB), Public Subnets (for NAT Gateway), Internet Gateway, Security Groups 
- Amazon EFS filesystem for Netskope Cloud Exchange
- NetskopeCloudExchangeTaskRole and NetskopeCloudExchangeTaskExecutionRole AWS IAM roles 
- Amazon ECS Task Definition
- Amazon ECS Fargate Cluster for deploying Netskope Cloud Exchange
- Amazon Application Load Balancer (optional)

## Service Quotas

The solution deploys a number of resources on your AWS account. You may need to consider service quotas (formerly known as service limits) on your account and increase them accordingly. Please refer to the table representing the number of resources created by the solution and review the service quotas following the service quotas documentation for each service. Please see the [AWS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html) for more details.

For example, the solution deploys 4 container instances on your Amazon ECS cluster. The non-adjustable quota for the number of container instances per cluster is 5000. Consider this limit when choosing the ECS cluster to host Netskope Cloud Exchange. You can find more details about Amazon ECS service quotas [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html)

|Resource|Resource count|Service Quotas references|
| :- | :- | :- |
|AWS IAM Role|2|[AWS IAM service quotas](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html)|
|AWS IAM Policy|1|[AWS IAM service quotas](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-quotas.html)|
|Amazon EFS File System|1|[Amazon EFS service quotas](https://docs.aws.amazon.com/efs/latest/ug/limits.html)|
|Amazon EFS Mount Target|2|[Amazon EFS service quotas](https://docs.aws.amazon.com/efs/latest/ug/limits.html)|
|Amazon EFS Access Point|4|[Amazon EFS service quotas](https://docs.aws.amazon.com/efs/latest/ug/limits.html)|
|Amazon CloudWatch Log Group|1|[Amazon CloudWatch service quotas](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_limits.html)|
|Amazon VPC|1|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Amazon Subnets|4|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Elastic IP addresses|2|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|NAT gateways|2|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Internet gateways|1|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Route Table|4|[Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Application Load Balancer|1|[Application Load Balancers quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Target group|1|[Application Load Balancers quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)|
|Amazon EC2 Security Group|3|[Amazon EC2 service quotas](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-resource-limits.html)|
|Amazon ECS Service|1|[Amazon ECS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html)|
|Amazon ECS Task|1|[Amazon ECS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html)|
|Amazon ECS Cluster|1|[Amazon ECS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html)|
|Amazon ECS Container Instance|4|[Amazon ECS service quotas](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-quotas.html)|


## Architecture Diagram
![](./media/NETSKOPE-CE-Architecture-Diagram.png)

*Fig 1. Netskope Cloud Exchange on AWS Fargate for Amazon ECS*

## Prerequisites 
The following prerequisites are required to implement the Netskope Cloud Exchange on AWS Fargate for Amazon ECS.

- This solution guide assumes working knowledge of the AWS management console. We also recommend that you become familiar with the following AWS services. <br/>
&emsp;&emsp; - [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.html) <br />
&emsp;&emsp; - [Amazon ECS](https://aws.amazon.com/ecs/) <br />
&emsp;&emsp; - [AWS Fargate](https://aws.amazon.com/fargate/) <br />
&emsp;&emsp; - [Amazon VPC](https://aws.amazon.com/vpc/) <br />
&emsp;&emsp; - [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) <br />

## Deployment & Configuration Steps

Using the CloudFormation template you can deploy the Netskope Cloud Exchange in two ways. Before going through the actual stack deployment process, it is highly recommended for the user to visit the [Best Practices](#best-practices-dos-and-donts) section.


1. Deploy both Infrastructure + Netskope Cloud Exchange using the AWS CloudFormation
2. Customize Infrastructure with existing resources (VPC, Subnets, NAT Gateway, ALB, ECS Cluster, etc.) and deploy Netskope Cloud Exchange using AWS CloudFormation

### 1. Deploy both Infrastructure + Netskope Cloud Exchange using the AWS CloudFormation

If you have your existing resources available, refer [ Customize Infrastructure with existing resources (VPC, Subnets, NAT Gateway, ALB, ECS Cluster, etc.) and deploy Netskope Cloud Exchange using AWS CloudFormation](#2-customize-infrastructure-with-existing-resources-vpc-subnets-nat-gateway-alb-ecs-cluster-etc-and-deploy-netskope-cloud-exchange-using-aws-cloudformation)

1.1 Download the **[CloudExchangeTemplate.yaml](https://github.com/netskopeoss/Netskope-CloudExchange-Amazon-ECS-Fargate/blob/main/CloudExchangeTemplate.yaml).**

1.2 Deploy the CloudFormation Stack on the AWS Management account.

1.2.1 Sign in to the AWS Management Console.

1.2.2 Navigate to the AWS CloudFormation management console and choose the region you'd like to deploy the automation solution.

1.2.3 Click on **Create stack** and choose **With new resources (standard).**

![](./media/NETSKOPE-CE-CFT-MGMT-Console.png)

1.2.4 Choose **Upload a template file** then click on **Choose file**. Choose the [CloudExchangeTemplate.yaml](https://github.com/netskopeoss/Netskope-CloudExchange-Amazon-ECS-Fargate/blob/main/CloudExchangeTemplate.yaml)  from the directory on your disk where you downloaded it, click **Open**, and then click **Next**.<br />

![](./media/NETSKOPE-CE-Stack-Upload-Template.png)

1.2.5 Enter the Stack name.

![](./media/NETSKOPE-CE-Stack-Name.png)

1.2.6 In the Parameters section select **True** in **Create new resources?** and provide an appropriate value for **Environment name**.<br />

![](./media/NETSKOPE-CE-Stack-Create-Resources-True.png)

1.2.7 In the **Application Load Balancer details** section Select **True** in **Create Application Load Balancer?** if you want to create ALB or Select **False** if you don't want to create.

*Notes* <br/>
*  *If you select **True** in **Create Application Load Balancer?**, then following implementations are applicable.* <br/>
    * *Access logs for ALB will be enabled.* <br/>
    * *S3 bucket and a respective bucket policy will be created for storing access logs of ALB. The name of ALB S3 bucket will be assigned as per the format given below.* <br/>
    *Format: nce-REGION-STACK_ID* <br/>
    *Example: nce-us-west-2-24c538a0-244d-11ed-9b54-025f9a3a09a6* <br/>
	* *S3 logging bucket and a respective logging bucket policy will be created for storing server access logs of above-created ALB S3 bucket. The name of S3 logging bucket will be automatically assigned by the CloudFormation.* <br/>
	*Format: STACK_NAME-loggingbucket-RANDOM_ALPHANUMERIC_STRING* <br/>
	*Example: nce-loggingbucket-1aln42m8gv711* <br/>
    * *You can view logs in both these S3 buckets after stack creation.* <br/>
	* *Post stack creation, for security reasons, it is recommended to enable bucket notifications manually for both the buckets created above by CloudFormation - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-11).* <br/>
* *If you select **False** in **Create Application Load Balancer?**, then following implementations are applicable.* <br/>
  * *If you have selected **False** in **Create Application Load Balancer?** and **Existing ARN of target group for ALB** field is **Blank** - No ALB will be created.* <br/>
  * *If you have selected **False** in **Create Application Load Balancer?** and **Existing ARN of target group for ALB** field is **Provided** - Existing ALB will be Used.* <br/>
    * *To find ARN of target group for ALB, please follow - [Finding ARN of target group for ALB](#finding-arn-of-target-group-for-alb).* <br/>
    * *In this case, it is recommended to enable access logs and create S3 bucket and respective bucket policy to store logs of ALB. It is also recommended to enable the server access logs in a logging bucket and an associated logging bucket policy for the created ALB S3 bucket - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-9). Post stack creation, for security reasons, it is also recommended to enable bucket notifications manually for both these buckets for security reasons - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-11).*

![](./media/NETSKOPE-CE-Stack-ALB-Details.png)

1.2.8 Skip the **Existing resource details(applicable if 'Create new resources?' is 'False')** section and navigate to the **Network details (applicable if 'Create new resources?' is 'True').**

Enter the VPC name and CIDR Ranges for VPC, Private, and Public Subnets according to your network requirements, or leave the default details as it is.

*Notes* <br/>
* *For security reasons, we have implemented in a way that,*
  *  *The default Network ACL will allow ingress for all the traffic other than port 22 and 3389 from 0.0.0.0/0.*
  * *The VPC default security group will not allow any inbound and outbound traffic.*

![](./media/NETSKOPE-CE-Stack-NW-Details.png)

1.2.9 Enter the Environment variables required for Netskope CE Deployment and click on **Next**. <br/><br/>
**Environment variables** <br/>
* JWT secrets: A random secure string that will be used for signing the authentication tokens.
* Maintainance password: A maintenance password that will be used for RabbitMQ and MongoDB services (This password can be set only once).
* Maintenance password escaped: URL encoded version of the Maintenance Password.
* TLS version: The TLS version that will be used to access the UI (TLSv1.3, TLSv1.2 TLSv1.3).
* HTTP proxy: Proxy URL to be used for the outbound HTTP traffic.
* HTTPS proxy: Proxy URL to be used for the outbound HTTPS traffic.
* CIDR to allow access to Netskope Cloud Exchange: IP range to grant access for Netskope Cloud Exchange (*required if load balancer is to be used*).

![](./media/NETSKOPE-CE-ENV-VARS.png)

1.2.10 In the **Configure stack options** section , specify tags (key-value pairs) to apply to resources in your stack. You can add up to 50 unique tags for each stack.

![](./media/NETSKOPE-CE-Stack-Tags.png)

1.2.11 In the **Permissions** section , choose an IAM role to explicitly define how CloudFormation can create, modify, or delete resources in the stack. If you don't choose a role, CloudFormation uses permissions based on your user credentials.

![](./media/NETSKOPE-CE-Stack-Permissions.png)

1.2.12 Expand the **Notification options** in **Advanced Options**, select the specific SNS topic ARN from the dropdown and click on **Next**. <br/>
*Note - If you want to create new SNS topic, refer [Creating a new SNS topic while stack creation.](#creating-a-new-sns-topic-while-stack-creation) This step is optional but recommended as per the best practices.* 

![](./media/NETSKOPE-CE-Stack-Advanced-Options-Exisiting-ARN.png)

1.2.13 Review the Stack details.

![](./media/NETSKOPE-CE-Stack-Review.png)

![](./media/NETSKOPE-CE-Stack-Create_Resources_List.png)

1.2.14 Select the checkbox of acknowledgment of IAM resources and click on **Create stack**.

![](./media/NETSKOPE-CE-Stack-ACK.png)

1.2.15 Wait for the Creation of Stack to be completed.

![](./media/NETSKOPE-CE-Stack-In-Progress.png)

After the successful creation, you can see the list of resources by selecting the **Resources** tab.

![](./media/NETSKOPE-CE-Stack-New-Resources-6.png)
![](./media/NETSKOPE-CE-Stack-New-Resources-5.png)
![](./media/NETSKOPE-CE-Stack-New-Resources-4.png)
![](./media/NETSKOPE-CE-Stack-New-Resources-3.png)
![](./media/NETSKOPE-CE-Stack-New-Resources-2.png)
![](./media/NETSKOPE-CE-Stack-New-Resources-1.png)


1.2.16 To access the application, refer to [Accessing Netskope CE](#accessing-netskope-ce).

*Note - Consider following before Accessing the Netskope CE*
* *For security reasons, Network ACL default Ingress from 0.0.0.0/0 for ports 22 and 3389 have been denied to block unauthorized access to the application via SSH and RDP. To allow them, the user needs to add a respective entry to ingress rules for Network ACL based on their use case.*

### 2. Customize Infrastructure with existing resources (VPC, Subnets, NAT Gateway, ALB, ECS Cluster, etc.) and deploy Netskope Cloud Exchange using AWS CloudFormation

2.1 Download the **[CloudExchangeTemplate.yaml](https://github.com/netskopeoss/Netskope-CloudExchange-Amazon-ECS-Fargate/blob/main/CloudExchangeTemplate.yaml)**.

2.2 Deploy the CloudFormation Stack on the AWS Management account.

2.2.1 Sign in to the AWS Management Console.

2.2.2 Navigate to the AWS CloudFormation management console and choose the region you'd like to deploy the automation solution.

2.2.3 Click on **Create Stack** and choose **With new resources (standard)**.

![](./media/NETSKOPE-CE-CFT-MGMT-Console.png)

2.2.4 Choose **Upload a template file** then click on Choose file. Choose the [CloudExchangeTemplate.yaml](https://github.com/netskopeoss/Netskope-CloudExchange-Amazon-ECS-Fargate/blob/main/CloudExchangeTemplate.yaml)  from the directory on your disk where you downloaded it to, click **Open** and then click **Next**.<br />

![](./media/NETSKOPE-CE-Stack-Upload-Template.png)

2.2.5 Enter the Stack name.

![](./media/NETSKOPE-CE-Stack-Name.png)

2.2.6 In the Parameters section select **False** in **Create new resources?** and provide an appropriate value for **Environment name**.<br />

![](./media/NETSKOPE-CE-Stack-Create-Resources-False.png)

2.2.7 In the **Application Load Balancer details** section Select **True** in **Create Application Load Balancer?** if you want to create ALB or Select **False** if you don't want to create.

*Notes* <br/>
*  *If you select **True** in **Create Application Load Balancer?**, then following implementations are applicable.* <br/>
    * *Access logs for ALB will be enabled.* <br/>
    * *S3 bucket and a respective bucket policy will be created for storing access logs of ALB. The name of S3 bucket will be assigned as per the format given below.* <br/>
    *Format: nce-REGION-STACK_ID* <br/>
    *Example: nce-us-west-2-24c538a0-244d-11ed-9b54-025f9a3a09a6* <br/>
	* *S3 logging bucket and a respective logging bucket policy will be created for storing server access logs of above-created ALB S3 bucket. The name of S3 logging bucket will be automatically assigned by the CloudFormation.* <br/>
	*Format: STACK_NAME-loggingbucket-RANDOM_ALPHANUMERIC_STRING* <br/>
	*Example: nce-loggingbucket-1aln42m8gv711* <br/>
    * *You can view logs in both these S3 buckets after stack creation.* <br/>
	* *Post stack creation, for security reasons, it is recommended to enable bucket notifications manually for both the buckets created above by CloudFormation - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-11).* <br/>
* *If you select **False** in **Create Application Load Balancer?**, then following implementations are applicable.* <br/>
  * *If you have selected **False** in **Create Application Load Balancer?** and **Existing ARN of target group for ALB** field is **Blank** - No ALB will be created.* <br/>
  * *If you have selected **False** in **Create Application Load Balancer?** and **Existing ARN of target group for ALB** field is **Provided** - Existing ALB will be Used.* <br/>
    * *To find ARN of target group for ALB, please follow - [Finding ARN of target group for ALB](#finding-arn-of-target-group-for-alb).* <br/>
    * *In this case, it is recommended to enable access logs and create S3 bucket and respective bucket policy to store logs of ALB. It is also recommended to enable the server access logs in a logging bucket and an associated logging bucket policy for the created ALB S3 bucket - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-9). Post stack creation, for security reasons, it is also recommended to enable bucket notifications manually for both these buckets for security reasons - [Ref](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp-controls.html#fsbp-s3-11).*

![](./media/NETSKOPE-CE-Stack-ALB-Details.png)


2.2.8 Provide the **Existing resources details (applicable if 'Create new resources?' is 'False')**.

*Notes* 
* *You have to enter the details of the resources that are available in your existing AWS infrastructure.*<br/>
* *Consider the following for AWS ECS Cluster.* <br/>
  * *If **Existing  ECS cluster name** field is blank - A new ECS Cluster will be created.*
  * *If **Existing  ECS cluster name** field is provided - It will check for the Cluster, if the Cluster is present, then it will be used as an existing resource or else it will throw the error at the time of stack creation.*
* *In the case of using existing VPC, for security reasons, the following implementations are optional but recommended.* 
  *  *Network ACLs should not allow ingress from 0.0.0.0/0 to port 22 or port 3389.*
  * *The VPC default security group should not allow any inbound and outbound traffic.*

![](./media/NETSKOPE-CE-Stack-Existing-Resources.png)

To Find Existing resources details please follow the details below.
* For VPC ID - [Finding a VPC ID](#finding-a-vpc-id) 
* For Subnet ID - [Finding a Subnet ID](#finding-a-subnet-id)
* For VPC CIDR block - [Finding a VPC CIDR block](#finding-a-vpc-cidr-block)
* For ECS Cluster Name - [Finding ECS Cluster Name](#finding-ecs-cluster-name)

2.2.9 Enter the Environment variables required for Netskope CE Deployment and click on **Next**. <br/><br/>
**Environment variables** <br/>
* JWT secrets: A random secure string that will be used for signing the authentication tokens.
* Maintainance password: A maintenance password that will be used for RabbitMQ and MongoDB services (This password can be set only once).
* Maintenance password escaped: URL encoded version of the Maintenance Password.
* TLS version: The TLS version that will be used to access the UI (TLSv1.3, TLSv1.2 TLSv1.3).
* HTTP proxy: Proxy URL to be used for the outbound HTTP traffic.
* HTTPS proxy: Proxy URL to be used for the outbound HTTPS traffic.
* CIDR to allow access to Netskope Cloud Exchange: IP range to grant access for Netskope Cloud Exchange (*required if load balancer is to be used*).

![](./media/NETSKOPE-CE-ENV-VARS.png)

2.2.10 In the **Configure stack options** section, specify tags (key-value pairs) to apply to resources in your stack. You can add up to 50 unique tags for each stack.

![](./media/NETSKOPE-CE-Stack-Tags.png)

2.2.11 In the **Permissions** section, choose an IAM role to explicitly define how CloudFormation can create, modify, or delete resources in the stack. If you don't choose a role, CloudFormation uses permissions based on your user credentials.

![](./media/NETSKOPE-CE-Stack-Permissions.png)

2.2.12 Expand the **Notification options** in **Advanced Options**, select the specific SNS topic ARN from the dropdown and click on **Next**. <br/>
*Note - If you want to create new SNS topic, refer [Creating a new SNS topic while stack creation.](#creating-a-new-sns-topic-while-stack-creation) This step is optional but recommended as per the best practices.*

![](./media/NETSKOPE-CE-Stack-Advanced-Options-Exisiting-ARN.png)

2.2.13 Review the Stack details.

![](./media/NETSKOPE-CE-Stack-Review.png)

![](./media/NETSKOPE-CE-Stack-Existing-Resources_List.png)

2.2.14 Select the checkbox of acknowledgment of IAM resources and click on **Create stack**.

![](./media/NETSKOPE-CE-Stack-ACK.png)


2.2.15 Wait for the Creation of Stack to be completed.

![](./media/NETSKOPE-CE-Stack-In-Progress.png)

After the successful creation, you can see the list of resources by selecting the **Resources** tab. <br/>

![](./media/NETSKOPE-CE-Stack-Existing-Resources-3.png)
![](./media/NETSKOPE-CE-Stack-Existing-Resources-2.png)
![](./media/NETSKOPE-CE-Stack-Existing-Resources-1.png)


2.2.16 To access the application, refer to [Accessing Netskope CE](#accessing-netskope-ce).

### Finding a VPC ID
Go to VPC Console, select **VPCs** and the relevant VPC. The VPC details page for the selected VPC opens with information including the VPC ID.

![](./media/NETSKOPE-CE-VPC-ID-Details.png)

### Finding a Subnet ID
Go to VPC Console, select **VPCs** and the relevant VPC. The VPC details page for the selected VPC opens with a table of subnets, click a subnet ID to open the details page and find the Subnet ID.

![](./media/NETSKOPE-CE-Subnet-ID-Details.png)

### Finding a VPC CIDR block
Go to VPC Console, select **VPCs** and the relevant VPC. The VPC details page for the selected VPC opens with information including the CIDRs section.

![](./media/NETSKOPE-CE-VPC-ID-Details.png)

### Finding ARN of target group for ALB
Go to EC2 console, select **Target Groups** and relevant Target Group. The Target Group details page for the selected Target Group opens the information including the ARN of Target Group.

![](./media/NETSKOPE-CE-Target-Group-ARN-Details.png)

### Finding ECS Cluster Name
Go to ECS Console, select **Clusters** and relevant Cluster. The Cluster details page for the selected Cluster opens the information including the Cluster Name.

![](./media/NETSKOPE-CE-ECS-Cluster-Details.png)

### Creating a new SNS topic while stack creation

1. Expand the **Notification options** in **Advanced Options** and click on **Create new SNS topic**.

![](./media/NETSKOPE-CE-Stack-Advanced-Options-Create-SNS-Topic1.png)

2. Provide **SNS topic name** and **Email address** and click on **Create SNS topic**.

![](./media/NETSKOPE-CE-Stack-Advanced-Options-Create-SNS-Topic2.png)


## Accessing Netskope CE 

**Access Netskope Cloud Exchange through Jumphost**

1. Go to EC2 Console, select **Launch instance**.

![](./media/NETSKOPE-CE-BH-Launch.png)

2. Provide the name for an instance and select Application and OS Image for instance.

![](./media/NETSKOPE-CE-BH-NAME-OS.png)

3. Select instance type and key pair to SSH into it.<br/>
*Note - Create new key pair if you are not having any key pair for login*.

![](./media/NETSKOPE-CE-BH-Key-Pair.png)

4. Edit the network settings and provide the details.

*Notes* 
* *VPC for instance and your CloudFormation resources must be same.*
* *Select the one of the public subnets to launch the instance.*
* *Select **Enable** the **Auto-assign public IP** to assign public IP.*

![](./media/NETSKOPE-CE-BH-NW-Setting.png) 

5. Click on **Launch instance** and leave the other details as default. Go to EC2 Console and wait for **Instance state** of your instance to become **Running**. <br/>

![](./media/NETSKOPE-CE-BH-Running.png)

6. Select the instance and Click on **Connect** which will open **Connect to instance** page. Click on **SSH Client** tab and copy the SSH command to the clipboard.

![](./media/NETSKOPE-CE-BH-SSH.png)

6. Go to EC2 Console, Select **Security Groups** and relevant security group. Click on **Edit inbound rules** and allow 8000 port.

![](./media/NETSKOPE-CE-BH-SG.png)

7. Go to VPC Console, Select **Network ACLs** and relevant Network ACL. Select the **Inbound rules** tab. Click on **Edit inbound rules** and add new rule for SSH from user's local machine.<br/>
*Note - Add your local machine IP Address in source tab.*

![](./media/NETSKOPE-CE-AA-NACL.png)

8. Go to EC2 Console, Select **Load Balancers** and relevant ALB. The ALB details page for the selected ALB opens the information including the DNS Name of ALB. Copy the DNS Name of ALB to the clipboard.

![](./media/NETSKOPE-CE-ALB-Details.png)


9. Open Git Bash or terminal of end user's local system and paste the following command that initiates the port forwarding from the local system to AWS EC2 Instance.<br/>
*Note - If ALB was not used in deployment, go to 9.2 Applocation Load Balancer is not present.*

   **9.1 Application Load Balancer is present** <br/>
  **Command:**   ssh -L LOCAL_HOST_PORT:ALB_DNS_NAME:LOAD_BALANCER_PORT  <br/>
  **Example:** ssh -i "test-demo.pem" ubuntu@ec2-54-193-33-60.us-west-1.compute.amazonaws.com -L 8000:internal-NetskopeCELB-netskope-ce.us-west-1.elb.amazonaws.com:80
  ![](./media/NETSKOPE-CE-Port-Forward.png)

   **9.2 Application Load Balancer is not present** <br/>
   **Command:**   ssh -L LOCAL_HOST_PORT:FARGATE_TASK_PRIVATE_IP:3000 <br/>
   **Example:**   ssh -i "test-demo.pem" ubuntu@ec2-54-193-33-60.us-west-1.compute.amazonaws.com -L 8000:172.16.2.86:3000<br/>
   *Note - To find Private IP of Task created in Fargate Cluster, refer to [Finding Private IP of Task in Fargate Cluster](#finding-private-ip-of-task-in-fargate-cluster).*
  ![](./media/NETSKOPE-CE-Port-Forward-without-ALB.png)


10. Type *localhost:8000* in browser and access the application.

![](./media/NETSKOPE-CE-App-Localhost-Login.png)

### Finding Private IP of Task in Fargate Cluster
Go to ECS Console, select **Clusters** and relevant Cluster. The Cluster details page for the selected Cluster opens the information. Click on **Tasks** tab and select the relevant task.

![](./media/NETSKOPE-CE-BH-Cluster_Task.png)

After selecting the relevent task, it opens the information including the private IP.

![](./media/NETSKOPE-CE-BH-Task-Private-IP.png)

## Best Practices (Dos and Don'ts)
*Note - These steps are optional but recommended to be followed manually before or after the creation of the stack depending upon the step.* <br/>
<div style="text-align: justify">

1. (Before Stack Creation) SNS topics should be encrypted at-rest using AWS KMS.<br/> Please follow the below steps for encrypting the SNS topic.<br/>
1.1 Go to SNS Console, click on **Topics** in navigation pane.<br/>
![](./media/NETSKOPE-CE-BP-SNS-Console.png)
1.2 Choose the name of the topic to encrypt. The SNS topic details page for the selected topic opens with information including the subscription, access policy, etc. Click on **Edit**.<br/>
![](./media/NETSKOPE-CE-BP-SNS-Topic1.png)
1.3 Expand the **Encrytion** section, choose **Enable Encryption**. Select the existing Customer master key (CMK) and click on **Save changes**. <br/>
*Note - If you are not having any Customer master key (CMK) then select default key*.
![](./media/NETSKOPE-CE-BP-SNS-Topic2.png)

2. (Before Stack Creation) Logging of delivery status should be enabled for notification messages sent to a topic.<br/>
Please follow the below steps for enabling the logging of delivery status.<br/>
2.1 Go to SNS Console, click on **Topics** in navigation pane.<br/>
![](./media/NETSKOPE-CE-BP-SNS-Console.png)
2.2 Choose the name of the topic to encrypt. The SNS topic details page for the selected topic opens with information including the subscription, access policy, etc. Click on **Edit**.<br/>
![](./media/NETSKOPE-CE-BP-SNS-Topic1.png)
2.3 Expand the **Delivery status logging** section. Fill up the information given below and click on **Save changes**.<br/>
    * Choose the protocol for which you want to log delivery status; for example **AWS Lambda**. 
    * Enter the **Success sample rate** (the percentage of successful messages for which you want to receive CloudWatch Logs.
    * In the **IAM roles** section, do one of the following:<br/>
      * To choose an existing service role from your account, choose **Use existing service role** and then specify IAM roles for successful and failed deliveries.
      * To create a new service role in your account, choose **Create new service role**, choose **Create new roles** to define the IAM roles for successful and failed deliveries in the IAM console. <br/>
      To give Amazon SNS write access to use CloudWatch Logs on your behalf, choose Allow.<br/>
      
    ![](./media/NETSKOPE-CE-BP-SNS-Delivery-Status-Logging.png)

3. (Before or After Stack Creation) Ensure a support role has been created to manage incidents with AWS Support.<br/>
Please refer the below link to ensure that a support role has been created to manage incidents with AWS Support.<br/>
    * Reference: https://docs.aws.amazon.com/console/securityhub/standards-cis-1.20/remediation <br/>

4. (After Stack Creation) Application and Classic Load Balancers logging should be enabled.<br/>
Please follow the below steps for enabling the logging.<br/>
4.1 Go to EC2 console, select **Load Balancers** and relevant Load balancer. click on **Actions** and click on **Edit attributes**.<br/>
![](./media/NETSKOPE-CE-BP-ALB-Edit-Attributes.png)
4.2 Select the checkbox of **Access logs** and provide the location of existing S3 bucket. Close the dialogue box.<br/>
*Note - If you do not have existing S3 bucket, select the checkbox of **Create this location for me***.
![](./media/NETSKOPE-CE-BP-ALB-Enable-Access-Logs.png)

5. (After Stack Creation) For the best security practices, we do not recommend assigning a public IP address to the Netskope Cloud Exchange service, but rather accessing it using a private IP address either via a jump host, Amazon Direct Connect, or [Netskope Private Access](https://www.netskope.com/products/private-access) (NPA).

6. (After Stack Creation) When first installed, Cloud Exchange does not require an SSL certificate and the web server can be reached over an unencrypted connection. You can either access front-end Netskope Cloud Exchange with [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) and deploy an SSL certificate there, or install a private SSL certificate on the Netskope Cloud Exchange. To learn how to install a private SSL certificate on the Netskope Cloud Exchange please see the documentation [here](https://docs.netskope.com/en/install-cloud-exchange.html). To learn how to create an HTTPS listener and install an SSL certificate on the Application Load Balancer, please follow [this](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html) documentation. 

7. (After Stack Creation) Manually created resources must be deleted before deleting the stack, as CloudFormation will only delete the resources created using a template file.

8. (After Stack Creation) Application Load Balancer deletion protection should be enabled.<br/>
Please follow below steps for enable deletion protection in ALB.<br/>
8.1 Go to EC2 console, select **Load Balancers** and relevant Load balancer. click on **Actions** and click on **Edit attributes**.<br/>
![](./media/NETSKOPE-CE-BP-ALB-Edit-Attributes.png)
8.2 Select the checkbox of **Deletion protection** and close the dialogue box.
![](./media/NETSKOPE-CE-TS-ALB-Remove-Deletion-Protection.png)

</div>

## Basic Troubleshooting
<div style="text-align: justify">

1. There is a default limit of 5 Elastic IP addresses per Region per AWS account. For more information about limits and how to request an increase, see [Elastic IP address limit](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-limit)
2. Don't provide long resource name - When the stack is created using CloudFormation, prefix or suffix will be added as per the stack name in resource's name. It will give error if the eventual resource name turns out to be too long.
3. If using existing load balancer, please make sure 80 port is not being used by another service, it will be used for accessing Netskope Cloud Exchange service.
4. Before deleting the stack, please remove the deletion protection in ALB, ohterwise ALB will not be deleted while deleting the stack.<br/>
Please follow below steps for removing deletion protection in ALB.<br/>
4.1 Go to EC2 console, select **Load Balancers** and relevant Load balancer. click on **Actions** and click on **Edit attributes**.<br/>
![](./media/NETSKOPE-CE-BP-ALB-Edit-Attributes.png)
4.2 Deselect the checkbox of **Deletion protection** and close the dialogue box.
![](./media/NETSKOPE-CE-TS-ALB-Remove-Deletion-Protection.png)

</div>

## FAQs

**Problem Statement** <br>
If you get following error -  *"STOPPED (ResourceInitializationError: unable to pull secrets or registry auth: execution resource retrival failed: unable to retrieve ECR registry auth: service call has been retried 3 time(s)."*

**Root Cause** <br/>
ECS Fargate cannot connect Netskope AWS ECR outside VPC.

This may occur when the VPC and Subnet do not have outside connectivity; hence, ECS Farget cannot connect to Netskope AWS ECR while pulling the container images for Netskope CE. 

**Solution** <br/>
The VPC and Subnet should have outside connectivity so that ECS Fargate can connect to Netskope AWS ECR. Please check whether the following entries are present in the Route Table.<br/>

![](./media/NETSKOPE-CE-FAQ1-Route-Table-Entry.png)

To ensure the outside connectivity exists or not, refer the below link. <br/>

* Reference: https://aws.amazon.com/premiumsupport/knowledge-center/ecs-unable-to-pull-secrets/




**Deploying Netskope Cloud Exchange using AWS ECS Fargate**

The Netskope Cloud Exchange (CE) provides customers with powerful integration tools to leverage investments across their security posture.

Cloud Exchange consumes valuable Netskope telemetry and external threat intelligence and risk scores, enabling improved policy implementation, automated service ticket creation, and exportation of log events from the Netskope Security Cloud.

To learn more about Netskope Cloud Exchange please refer to the [Netskope Cloud Exchange introduction page](https://www.netskope.com/products/capabilities/cloud-exchange).

Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that makes it easy for you to deploy, manage, and scale containerized applications.

AWS Fargate is a serverless, pay-as-you-go compute engine that lets you focus on building applications without managing servers. AWS Fargate is compatible with Amazon Elastic Container Service (ECS). To learn more about Amazon ECS please follow the [Amazon ECS](https://aws.amazon.com/ecs/) documentation. To learn more about AWS Fargate please follow the [AWS Fargate](https://aws.amazon.com/fargate/) documentation page.

This document will guide you how to deploy Netskope Cloud Exchange using AWS Fargate on Amazon ECS.

This solution consists of the following components:

CloudExchangeTemplate.yaml AWS CloudFormation template that deploys the following resources:

- Amazon EFS filesystem for Netskope Cloud Exchange
- Custom resource AWS Lambda function to create initial directory structure on the Amazon EFS filesystem above
- NetskopeCloudExchangeTaskRole and NetskopeCloudExchangeTaskExecutionRole AWS IAM roles 
- NetskopeCETaskSecurityGroup will be used by the Netskope CE task

CloudExchangeTaskDefinition.json Task Definition json file used to create a new task definition for the Netskope Cloud Exchange

Architecture diagram

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.001.png)

*Fig 1. Netskope Cloud Exchange on AWS Fargate for Amazon ECS*

Prerequisites 

The following prerequisites are required to implement the Netskope Cloud Exchange on AWS Fargate for Amazon ECS:

- Existing Amazon VPC with minimum of two subnets in different Availability Zones and outbound connectivity to the Netskope NewEdge platform, third party applications and partners’ platforms you’re planning to integrate Netskope Cloud Exchange with, as well as with regional endpoint for Amazon S3 for the Custom Resource AWS Lambda function to report it status to Amazon CloudFormations. To enable Netskope Cloud Exchange communicating with external third-party services we recommend deploying [Amazon Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) in your VPC. 
- Existing Amazon ECS cluster where Netskope Cloud Exchange task will be running. To learn how to work with Amazon ECS please refer to the [tutorials here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-tutorials.html). 
- The latest version of the AWS CLI is installed and configured. For more information about installing or upgrading your AWS CLI, see [Installing the AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
- This solution guide assumes working knowledge with the AWS management console and AWS CLI. We also recommend that you become familiar with the following AWS services:
&emsp;&emsp; - [AWS Lambda](https://aws.amazon.com/lambda/)
&emsp;&emsp; - [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/)
&emsp;&emsp; - [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/GettingStarted.html)
&emsp;&emsp; - [Amazon VPC](https://aws.amazon.com/vpc/)
&emsp;&emsp; - [AWS CLI](https://aws.amazon.com/cli/)


# Deployment and Configuration Steps
## Deploying Amazon EFS Filesystem, AWS Custom resource Lambda function, IAM roles and Netskope CE Task Security group using CloudExchangeTemplate.yaml and creating Netskope Cloud Exchange Task Definition.
Download the CloudExchangeTemplate.yaml and CloudExchangeTaskDefinition.json to your computer.

**Step 1.1:** **Deploy the CloudFormation Stack on the AWS Security Management account**

Sign into the AWS Security Management account as administrator and deploy the Netskope Cloud Exchange CloudFormations stack. 

1.1.1. Navigate to the AWS CloudFormation management console and choose the region you’d like to deploy the automation solutions in the Security Management account. 
1.1.2. Click **Create Stack** and choose **With new resources (standard)**.

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.002.png)

1.1.3. Choose **Upload a template file** then click on Choose file. Choose the CloudExchangeTemplate.yaml from the directory on your disk where you downloaded it to, click **Open** and then click **Next**.

![Graphical user interface, text, application, email

Description automatically generated](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.003.png)

1.1.4. Enter the stack name and the parameters for your deployment:

|Existing VPC Id|Enter the existing VPC Id where Netskope Cloud Exchange will be deployed|
| :- | :- |
|Existing Private Subnet ID 1|Enter the first Subnet ID where the EFS filesystem for Netskope Cloud Exchange will be deployed. Note that custom resource Lambda function in this stack should be able to communicate from this subnet to Amazon S3 regional endpoint|
|Existing Private Subnet ID 2 |Enter the second Subnet ID where the EFS filesystem for Netskope Cloud Exchange will be deployed. Note that custom resource Lambda function in this stack should be able to communicate from this subnet to Amazon S3 regional endpoint|

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.004.png)


1.1.5. Click **Next**.
1.1.6. Optionally, enter the Tags for your CloudFormation stack and / or click Next.

![Graphical user interface, application

Description automatically generated](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.005.png)

1.1.7. Acknowledge creating IAM resources and click **Create stack**.

![Graphical user interface, text, application

Description automatically generated](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.006.png)

1.1.8. When CloudFormation stack is in the CREATE\_COMPLETE state, navigate to the Output tab and see the Security Group Id you will need to customize to allow Netskope Cloud Exchange access to the Netskope NewEdge and third-party platforms.
#### Please follow the instructions in the [IP Allowlisting](https://docs.netskope.com/en/ip-allowlisting.html) article on the Netskope Knowledge Portal to add the Netskope NewEdge IP addresses to the Netskope CE Task Security Group.


![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.007.png)

1.1.9. Note the EFS File System Id in the CloudFormation  stack output. It’ll be used late on while creating the Netskope Cloud Exchange Task Definition for Amazon ECS.

**Step 1.2: Create Netskope Cloud Exchange Task Definition:**

1.2.1. Open the CloudExchangeTaskDefinition.json file for editing and replace all occurrences of the following values as following:

|/\*AWS Region\*/|AWS Region where you’re deploring the AWS Cloud Exchange. For example, us-east-2|
| :- | :- |
|/\*EFS FS Id\*/|AWS EFS File System Id created in by the CloudFormation template above and recorder in the step 1.1.9 above.|
|<p>/\*AWS Account ID\*/</p><p></p>|AWS Account ID where Netskope Cloud Exchange been deployed|



1.2.2. Using AWS CLI, create a new Amazon ECS Task Definition for Netskope Cloud Exchange:

aws ecs register-task-definition --family NetskopeCloudExhange3-0 --cli-input-json file://CloudExchangeTaskDefinition.json


## Deploying Netskope Cloud Exchange AWS Fargate task on Amazon ECS.
You can use the Netskope Cloud Exchange Task Definition to deploy AWS Fargate Task or Service according to your organization’s best practices using your existing automation tools. 


**Step 2.1:** **Run Netskope Cloud Exchange as a service using AWS Fargate on Amazon ECS**


2.1.1. Sign into Amazon ECS management console and navigate to Task Definitions and choose NetskopeCloudExchange3-0 task

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.008.png)

2.1.2. Click on Actions->Create Service

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.009.png)

2.1.3. Set the following configuration for the new service:


|<p>**Launch type**</p><p></p>|Fargate|
| :- | :- |
|<p>**Operating system family**</p><p></p>|Linux|
|<p>**Platform version**</p><p></p>|LATEST|
|<p>**Cluster**</p><p></p>|Choose the Amazon ECS cluster you’d like to run Netskope Cloud Exchange on|
|<p>**Service name**</p><p></p>|NetskopeCloudExchange (you can choose the name according to your preferences and best practices)|
|<p>**Service type**</p><p></p>|REPLICA|
|<p>**Number of tasks**</p><p></p>|1|
|<p>**Minimum healthy percent**</p><p></p>|100|
|<p>**Maximum percent**</p><p></p>|200|
|<p>**Deployment circuit breaker**</p><p></p>|Disabled|

Leave other parameters unchanged and click Next step

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.010.png)

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.011.png)

2.1.4. Choose the same VPC and VPC Subnets you used to create an EFS File System using CloudExchangeTemplate.yaml.

Choose the security group you noted in the step 1.1.8 above and click Next step.

For the best security practices, we do not recommend assigning public IP address to the Netskope Cloud Exchange service, but rather accessing it using private IP address either via a jump host, Amazon Direct Connect or [Netskope Private Access](https://www.netskope.com/products/private-access) (NPA).

When first installed, Cloud Exchange does not require an SSL certificate and the web server can be reached over an unencrypted connection. You can either front-end Netskope Cloud Exchange with [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) and deploy an SSL certificate there, or install a private SSL certificate on the Netskope Cloud Exchange. To learn how to install a private SSL certificate on the Netskope Cloud Exchange please see the documentation [here](https://docs.netskope.com/en/install-cloud-exchange.html). To learn how to create an HTTPS listener and install SSL certificate on the Application Load Balancer, please follow [this](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html) documentation. 

To enable Netskope Cloud Exchange communicating with external third-party services we recommend deploying [Amazon Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html). 




![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.012.png)

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.013.png)

2.1.5. Leave Set Auto Scaling unchanged and click Next step

![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.014.png)

2.1.6. Review the configuration and click Create service
2.1.7. To monitor the task status, navigate to Clusters->your ECS cluster->Tasks menu


![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.015.png)

and click on the NetskopeCloudExchange task.

Wait till the task status will be RUNNING with all four containers HEALTHY.



![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.016.png)

Now you can use the task Private IP address to sign into the Netskope Cloud Exchange. 

After initial installation Netskope Cloud Exchange is available via HTTP and you can access it via http://*<Task IP address>.* To secure access to the Netskope Cloud Exchange follow the documentation mentioned in the step 1.2.6 above.

For complete documentation on how to use Netskope Cloud Exchange please refer to the [Netskope Cloud Exchange documentation](https://docs.netskope.com/en/netskope-cloud-exchange.html). 
![](.//media/NetskopeCE.09fc7e95-8fc0-4a42-9c93-89f7a36aafbe.017.png)


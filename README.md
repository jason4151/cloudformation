# CloudFormation

AWS CloudFormation templates used as a starting point for deploying an assortment of services.

## VPC

The example VPC `vpc.yaml` template is comprised of a 10.x.0.0/24 primary CIDR block, giving 256 private IP addresses. This address space is then divided into eight 10.x.0.0/27 subnets, each with 32 private IP addresses (5 of which are automatically reserved by AWS). A unique number is chosen for the second octet which will allow for VPC peering and routing between subnets should the need ever arise.

To facilitate high availability and service isolation, each VPC is contains 2 subnets of each type spanning two Availability Zones (AZs). There are 2 Public (internet-facing) subnets containing 1 NAT Gateway each, 2 Management (private) subnets, 2 Product (private) & 2 Database (private) Subnets.

* The public subnets are intended for NAT Gateways, Load Balancers, Bastion hosts, etc.
* The management subnets are intended for internally used services, e.g., Active Directory, Jenkins, Nexus, etc.
* The product subnets are intended for product (demo) deployments.
* The database subnets are intended for RDS, Redshift, etc.

Any service requiring inbound internet access to a front-end service, should be configured to use a Load Balancer deployed into the Public Subnets. All outbound internet traffic is routed through one of the NAT Gateway devices in the respective AZ. A VPC Endpoint is also configured to allow certain types of traffic to stay within the VPC instead of traversing the internet, S3 being an example of this.

### Deployment

`aws cloudformation create-stack --stack-name vpc-19-us-east-1-demo --template-body file://vpc.yaml`

### VPC Diagram

![VPC Diagram](https://www.lucidchart.com/publicSegments/view/8017025b-b0a9-482f-819b-bd624e94c120/image.png)

## CodeDeploy & CodePipeline

The deployment and maintenance of application code is done through the use of CodeDeploy and the method for triggering the CodeDeploy job is done through the use of CodePipeline. For example, the GitHub repository for the demo web application [php-crud-rds](https://github.com/jason4151/php-crud-rds) uses two branches, master and test. The master branch corresponds to production code and deployments and the test branch corresponds to test code and deployments. Each of these branches is linked to their own EC2 instance, CodeDeploy job and CodePipeline configuration within AWS. The CodeDeploy and CodePipeline configuration is defined in the CloudFormation template `ec2-web-server.yaml`. Additional branches could be added depending on the specific use case, and each of those would likely correspond to a specific application stack.

## KMS

The `kms-service-config.yaml` template, while not deployable itself, demonstrates a working configuration for a number of AWS services with encryption at rest using KMS:

* DynamoDB
* Elasticsearch
* EMR
* Kinesis
* S3
* SQS

## EC2

Templates named `ec2-*.yaml` deploy the specified service utilizing an EC2 instance. Additional AWS services utilized within these templates are as follows:

* IAM
* S3
* CodeDeploy
* CodePipeline
* Secrets Manager
* Route 53

### Demo Web Application Diagram

The Demo Web application uses a combination of the following templates:

* `vpc.yaml`
* `rds-mysql.yaml` or `rds-aurora-mysql.yaml`
* `ec2-linux-bastion.yaml`
* `ec2-salt-master.yaml`
* `ec2-web-server.yaml`
![Demo Web App](https://www.lucidchart.com/publicSegments/view/d0c7a8ae-312e-4810-9101-95e95471aeb9/image.png)

### ECS

The `ecs.yaml` template creates an Amazon ECS cluster with auto scaling policies for a containerized application. The `ecs-fargate.yaml` template creates an ECS Fargate Serverless deployment using a containerized application. The `ecr.yaml` demonstrates the configuration of an Docker image repository with am IAM cross account access role. This allows other AWS accounts to access the repository.

### ECS Example Application Diagram

This diagram depicts an example ECS application deployment.
![ECS Example App](https://www.lucidchart.com/publicSegments/view/31db8182-a28c-486b-8d1f-803a5e6d89be/image.png)

### RDS

The `rds-aurora-mysql.yaml` template creates a RDS Aurora MySQL Database cluster in serverless mode. Using RDS serverless removes the complexity of managing database instances and capacity. The database will automatically start up, shut down, and scale to match the needs of the application. AWS only charges for the database resources consumed, on a per-second basis. In other words, you don't pay for the database instance unless it's actually running. In contrast, the `rds-mysql.yaml` template creates a single RDS MySQL Database instance which you will be charged for as long as the instance is running. Both templates utilize Secrets Manager for generating the database admin user password.

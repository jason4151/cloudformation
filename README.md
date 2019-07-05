# CloudFormation
AWS CloudFormation templates used as a starting point for deploying an assortment of services.

## Networking & Content Delivery
### VPC
The example VPC `vpc.yml` template is comprised of a 10.x.0.0/24 primary CIDR block, giving 256 private IP addresses. This address space is then divided into eight 10.x.0.0/27 subnets, each with 32 private IP addresses (5 of which are automatically reserved by AWS). A unique "Supernet" number (second octet) is chosen for each deployment which will allow for VPC peering and routing between subnets should the need ever arise.

To facilitate high availability and service isolation, each VPC is contains 2 subnets of each type spanning two Availability Zones (AZs). There are 2 Public (internet-facing) subnets containing 1 NAT Gateway each, 2 Management (private) subnets, 2 Product (private) & 2 Database (private) Subnets.

* The public subnets are intended for NAT Gateways, Load Balancers, Bastion hosts, etc.
* The management subnets are intended for internally used services, e.g., Active Directory, Jenkins, Nexus, etc.
* The product subnets are intended for product (demo) deployments.
* The database subnets are intended for RDS, Redshift, etc.

Any service requiring inbound internet access to a front-end service, should be configured to use a Load Balancer deployed into the Public Subnets. All outbound internet traffic is routed through one of the NAT Gateway devices in the respective AZ. A VPC Endpoint is also configured to allow certain types of traffic to stay within the VPC instead of traversing the internet, S3 being an example of this.

#### Deployment
`aws cloudformation create-stack --stack-name vpc-19-us-east-1-demo --template-body file://vpc.yml`

![VPC Diagram](https://www.lucidchart.com/publicSegments/view/8131766d-ceeb-4ca5-af5d-36ea8e7e14dd/image.png)

## Developer Tools
### CodeDeploy & CodePipeline
The deployment and maintenance of application code is done through the use of CodeDeploy and the method for triggering the CodeDeploy job is done through the use of CodePipeline. For example, the GitHub repository for the demo web application [php-crud-rds](https://github.com/jason4151/php-crud-rds) uses two branches, master and test. The master branch corresponds to production code and deployments and the test branch corresponds to test code and deployments. Each of these branches is linked to their own EC2 instance, CodeDeploy job and CodePipeline configuration within AWS. The CodeDeploy and CodePipeline configuration is defined in the CloudFormation template `ec2-web-server.yml`.

## Security, Identity, & Compliance
### KMS
The `kms-service-config.yml` template, while not deployable, demonstrates a working configuration of a number of AWS services with encryption at rest using KMS:
* DynamoDB
* Elasticsearch
* EMR
* Kinesis
* S3
* SQS

## Compute
### EC2
Templates named `ec2-*.yml` create the respective service utilizing an EC2 instance. Additional AWS services utilized within these templates:
* IAM
* S3
* CodeDeploy
* CodePipeline
* Secrets Manager
* Route 53

### ECS
The `ecs.yml` template creates an Amazon ECS cluster with auto scaling for an application. The `ecs-fargate.yml` template creates an ECS Fargate Serverless deployment using an example application.

## Database
### RDS
The `rds-aurora-mysql.yml` template creates a RDS Aurora MySQL Database cluster in Serverless mode. The `rds-mysql.yml` template creates a single RDS MySQL Database instance. Both templates utilize Secrets Manager for generating the database admin user password.

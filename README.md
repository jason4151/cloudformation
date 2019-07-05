# CloudFormation
AWS CloudFormation templates used as a starting point for deploying an assortment of services.

## Networking & Content Delivery
### VPC
The example VPC `vpc.yml` template is comprised of a 10.x.0.0/24 primary CIDR block, giving 256 private IP addresses. This address space is then divided into eight 10.x.0.0/27 subnets, each with 32 private IP addresses (5 of which are automatically reserved by AWS). A unique "Supernet" number (second octet) is chosen for each deployment which will allow for VPC peering and routing between subnets should the need ever arise.

To facilitate high availability and service isolation, each VPC is contains 2 subnets of each type spanning two Availability Zones (AZs). There are 2 Public (internet-facing) subnets containing 1 NAT Gateway each, 2 Management (private) subnets, 2 Product (private) & 2 Database (private) Subnets. The management subnet is intended for internally used services, e.g., Active Directory, Jenkins, Nexus, etc. The product subnet is intended for product (demo) deployments. The database subnet is intended for RDS, Redshift, etc. Any service requiring inbound internet access to a front-end service, should be configured to use a Load Balancer deployed into the Public Subnets. All outbound internet traffic is routed through one of the NAT Gateway devices in the respective AZ. A VPC Endpoint is also configured to allow certain types of traffic to stay within the VPC instead of traversing the internet, S3 being an example of this.

#### Deployment
`aws cloudformation create-stack --stack-name vpc-19-us-east-1-demo --template-body file://vpc.yml`

![VPC Diagram](https://www.lucidchart.com/publicSegments/view/8131766d-ceeb-4ca5-af5d-36ea8e7e14dd/image.png)

## Security, Identity, & Compliance
### KMS
The `kms-service-config.yml` template demonstrates the configuration of a number of AWS services with encryption at rest using KMS:
* DynamoDB
* Elasticsearch
* EMR
* Kinesis
* KMS
* S3
* SQS

## Compute
### EC2

### ECS
The `ecs.yml` template creates an Amazon ECS cluster with auto scaling for an application. The `ecs-fargate.yml` template creates an ECS Fargate "Serverless" deployment using an example application.

## Database
### RDS

## Developer Tools
### CodePipeline

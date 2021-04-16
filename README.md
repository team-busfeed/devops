# Dev/Ops files
This repository includes all the YAML files for CloudFormation. Please see the below illustration for how our cloud architecture is like:

![Project Busfeed Cloud Architecture Diagram](https://user-images.githubusercontent.com/54618162/115047754-64d3f080-9f0b-11eb-8bc1-a63e21748fd6.png)

> **_Note that these CloudFormation templates only account for the backend services on Fargate, the VPCs and associated subnets, security groups and ECR repositories. It excludes the EC2 used to run the polling script, RDS and S3 assets._**

## Order of files

When using CloudFormation, please follow the below sequence to kickstart the architecture deployment:

1. **Networking**: `network.yml`
2. **ECR Repositories**: `ecr.yml`
3. **Security Groups**: `sec_grps.yml`
4. **App stack**: `fargate.yml`

> 1 & 2 has no particular sequence/dependencies; however, 3 has dependencies from 1, and 4 has dependencies from 1 & 3.

## CloudFormation Designer

To visualise the architecture with ease, we've used the CloudFormation built-in Designer app that allows us to view the dependencies between each service, especially with `fargate.yml`. Hence the following portion of the file represents the metadata for CF's Designer:

```
Metadata:
  'AWS::CloudFormation::Designer':
  ...
```

## Network

The network file creates:
- 01 VPC
- 01 Internet Gateway
- 02 Public Subnets
- 01 Route Table
- 02 Route Table Associations
- 03 outputs
  - VPC-ID
  - PublicSubnetA
  - PublicSubnetB

## ECR Repositories

The ECR Repositories file initialises our repositories for each of the four (4) microservices on our backend. Note that one is able to perform this manually through the AWS Console, however for ease of deployment and maintainability, we opted to use CloudFormation. Here are the repositories it will create:

1. location-service
2. demand-service
3. beacon-service
4. pda-service

## Network

There's not much to dive into the security group settings here. It's pretty standard.

## Fargate

See below for a quick overview of the Fargate CloudFormation stack:

![Project Busfeed Fargate CloudFormation Stack](https://user-images.githubusercontent.com/54618162/115046182-c4c99780-9f09-11eb-8ade-ca2816443d1e.png)

Basically, for each service, a task is created. A CloudWatch log is attached to the task. Next, this service is then attached to a `TargetGroup`, and then a Load Balancer Listener Rule is attached to it. Finally, the rule is attached to the LB Listener that is dependent on the Load Balancer.

Each task contains the container specifications and size (RAM, vCPU, etc.) and the ECR repository to pull the container from. For autoscaling, it is not specified in this CloudFormation as we felt that it would be better to configure through AWS Console instead.

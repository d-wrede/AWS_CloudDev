# Stepthrough Overview

The task is to build a highly scalable and load balanced Web server infrastructure in two AZs.
## Resources:

- VPC
  - Route Table
    - route for internet gateway
    - route association for public subnets
  - Internet Gateway
  - optional: NAT gateway to public subnets
    A NAT gateway is used to enable instances in private subnets to communicate with the internet, for example for downloading software updates. In this exercise it may be optional.
- security groups
  - Elastic Load Balancer
  - EC2 WebApps
  - Databases
- S3 bucket
- 2 public subnets in each AZ
- 2 private subnets in each AZ
- EC2 groups as web servers in each public subnet
  - install wordpress and DB_client
  - connect to DB
- databases RDS
  - DBsubnet
  - DBcluster Aurora
  - DB Master in private subnet1, in AZ-A (write)
  - DB Secondary in private subnet2, in AZ-B (read replica)
- elastic load balancer for ec2 groups
- auto scaling system
  - for ec2 groups & DBs (only read replicas?)
  - launch template
  - target tracking scaling policy (dynamic)

- Install stress tool for testing
- Cost Estimate Calculator

## Approach

1. Create a VPC.
2. Create four subnets: two public subnets in each AZ and two private subnets in each AZ.
3. Create an Internet Gateway and attach it to the VPC.
4. Create security groups for the Elastic Load Balancer, EC2 WebApps, and Databases.
5. Launch EC2 instances in the public subnets, install WordPress and DB_client, and connect them to the databases. Use the appropriate security groups to allow traffic to flow between the different resources.
6. Create the RDS databases, including the DBsubnet and DBcluster Aurora, and configure the master database in the private subnet1 of AZ-A and the secondary read replica database in the private subnet2 of AZ-B.
7. Create an Elastic Load Balancer and configure it to balance traffic between the EC2 instances.
8. Set up an auto scaling system for both the EC2 instances and the databases.
9. Create a launch template for the EC2 instances.
10. Set up a target tracking scaling policy to automatically adjust the number of instances based on the current demand.
11. Create an auto scaling group for the read replicas, using a similar target tracking scaling policy to scale the read replicas based on demand.

## building process

Using <https://subnettingpractice.com/vlsm.html>

The network 10.0.0.0/24 has 254 hosts. Your subnets need 80 hosts.

| Name           | Hosts Needed | Hosts Available | Unused Hosts | Network Address | Slash | Mask            | Usable Range           | Broadcast  | Wildcard |
|----------------|--------------|-----------------|--------------|-----------------|-------|-----------------|------------------------|------------|----------|
| PublicSubnet1  | 20           | 30              | 10           | 10.0.0.0        | /27   | 255.255.255.224 | 10.0.0.1 - 10.0.0.30   | 10.0.0.31  | 0.0.0.31 |
| PublicSubnet2  | 20           | 30              | 10           | 10.0.0.32       | /27   | 255.255.255.224 | 10.0.0.33 - 10.0.0.62  | 10.0.0.63  | 0.0.0.31 |
| PrivateSubnet1 | 20           | 30              | 10           | 10.0.0.64       | /27   | 255.255.255.224 | 10.0.0.65 - 10.0.0.94  | 10.0.0.95  | 0.0.0.31 |
| PrivateSubnet2 | 20           | 30              | 10           | 10.0.0.96       | /27   | 255.255.255.224 | 10.0.0.97 - 10.0.0.126 | 10.0.0.127 | 0.0.0.31 |


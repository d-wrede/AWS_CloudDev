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
  - 2 public subnets, each in a different AZ
  - 2 private subnets, each in a different AZ
- security groups
  - Elastic Load Balancer
  - EC2 WebApps
  - Databases
- S3 bucket
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

Create:

1. a VPC.
   - four subnets: two public subnets in each AZ and two private subnets in each AZ.
2. an Internet Gateway and attach it to the VPC.
3. security groups for the Elastic Load Balancer, EC2 WebApps, and Databases.
4. Load Balancer:
   - a target group for your instances.
   - a load balancer (Application or Network Load Balancer, depending on your requirements).
   - listener
5. empty
6. an Auto Scaling group
   - a launch template with the desired EC2 instance settings (install WordPress and DB_client)
7. the RDS databases, including the DBsubnet and DBcluster Aurora, and configure the master database in the private subnet1 of AZ-A and the secondary read replica database in the private subnet2 of AZ-B.
8. an Elastic Load Balancer and configure it to balance traffic between the EC2 instances.
9. Set up an auto scaling system for both the EC2 instances and the databases.
10. a launch template for the EC2 instances.
11. Set up a target tracking scaling policy to automatically adjust the number of instances based on the current demand.
12. an auto scaling group for the read replicas, using a similar target tracking scaling policy to scale the read replicas based on demand.

## building process

Using <https://subnettingpractice.com/vlsm.html>

The network 10.0.0.0/24 has 254 hosts. Your subnets need 80 hosts.

| Name           | Hosts Needed | Hosts Available | Unused Hosts | Network Address | Slash | Mask            | Usable Range           | Broadcast  | Wildcard |
|----------------|--------------|-----------------|--------------|-----------------|-------|-----------------|------------------------|------------|----------|
| PublicSubnet1  | 20           | 30              | 10           | 10.0.0.0        | /27   | 255.255.255.224 | 10.0.0.1 - 10.0.0.30   | 10.0.0.31  | 0.0.0.31 |
| PublicSubnet2  | 20           | 30              | 10           | 10.0.0.32       | /27   | 255.255.255.224 | 10.0.0.33 - 10.0.0.62  | 10.0.0.63  | 0.0.0.31 |
| PrivateSubnet1 | 20           | 30              | 10           | 10.0.0.64       | /27   | 255.255.255.224 | 10.0.0.65 - 10.0.0.94  | 10.0.0.95  | 0.0.0.31 |
| PrivateSubnet2 | 20           | 30              | 10           | 10.0.0.96       | /27   | 255.255.255.224 | 10.0.0.97 - 10.0.0.126 | 10.0.0.127 | 0.0.0.31 |

## comments

During the building phase I corrected the approach above, to not launch an instance manually, or hardcoded into the json file, but use the AutoScaling group for that purpose. For the DB instances I decided a similar approach for the replicas, while the primary database is set up in the script.

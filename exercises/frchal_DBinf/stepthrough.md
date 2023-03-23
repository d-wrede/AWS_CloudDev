# Stepthrough
Following the descriptions here:
- https://github.com/Jan0770/terraformPractice/
- https://github.com/JonasHaas/nf-db-challenge/

## content
- vpc
  - 1 public subnet
  - 4 private subnets in different AZs. 2 for the db-subnet, 2 for replica
  - db-subnet
  - internet gateway
  - public route table
    - route to internet gateway
    - route table association / link to subnet

- ec2 instance / command host
  - ami           = "ami-0df24e148fdb9f1d8"
  - instance_type = "t3.micro"
  - subnet_id = aws_subnet.public_subnet.id
  - vpc_security_group_ids = [aws_security_group.allow_ssh.id, aws_security_group.allow_ec2_aurora.id]
  - key_name = "vockey"
  - associate_public_ip_address = true
  - tags = {  Name = "Aurora Access Instance"  }

- database 
  - RDS cluster Aurora
    - version: choose low, but not lowest. To match or 'get close to'  the client version (maria DB 5.5?). The lowest is not accessible in the sandbox. You'll receive an error regarding the IAM roles.
    - final snapshot?
    - delegate to private subnets
    - assign security group 'allow aurora access'
    - database_name
    - master_username
    - master_password
    - tag Name (for stack)
  - DB cluster instance
    - choose 2 for Multi-AZ
    - link DB instance to ec2 instance via endpoints

- security groups
  - Allow SSH traffic (ingress port 22, cidr 0.0.0.0/0)
  - allow web_server with aurora communication
    - port 3306 ingress
  - allow aurora all egress (outbound) communication
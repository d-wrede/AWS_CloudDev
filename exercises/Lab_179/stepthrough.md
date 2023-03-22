Lab: 179-[JAWS]-Activity - Migrate to Amazon RDS 

aws ec2 describe-instances \
--filters "Name=tag:Name,Values= CafeInstance" \
--query "Reservations[*].Instances[*].[InstanceId,InstanceType,PublicDnsName,PublicIpAddress,Placement.AvailabilityZone,VpcId,SecurityGroups[*].GroupId]"

[[[
            "i-02347e69026d68b65", 
            "t3.small", 
            "ec2-35-89-224-234.us-west-2.compute.amazonaws.com", 
            "35.89.224.234", 
            "us-west-2a", 
            "vpc-0fb54f9536bfbaf86", 
            [
                "sg-0bc4f0499b69584ea"
            ]
        ]]]

CafeInstance Instance ID: i-02347e69026d68b65
CafeInstance Instance Type: t3.small
CafeInstance Public DNS Name: ec2-35-89-224-234.us-west-2.compute.amazonaws.com
CafeInstance Public IP Address: 35.89.224.234
CafeInstance Availability Zone: us-west-2a
CafeInstance VPC ID: vpc-0fb54f9536bfbaf86
CafeSecurityGroup Group ID: sg-0bc4f0499b69584ea

Determine the IPv4 CIDR block:
aws ec2 describe-vpcs --vpc-ids vpc-0fb54f9536bfbaf86 \
--filters "Name=tag:Name,Values= Cafe VPC" \
--query "Vpcs[*].CidrBlock"

Cafe VPC IPv4 CIDR block: 10.200.0.0/20

Determine the Subnet ID and IPv4 CIDR block:
aws ec2 describe-subnets \
--filters "Name=vpc-id,Values=vpc-0fb54f9536bfbaf86" \
--query "Subnets[*].[SubnetId,CidrBlock]"

Cafe Public Subnet 1 Subnet ID: subnet-009d69dfd25e49081
Cafe Public Subnet 1 IPv4 CIDR block: 10.200.0.0/24

aws ec2 describe-availability-zones \
--filters "Name=region-name,Values=us-west-2" \
--query "AvailabilityZones[*].ZoneName"
[
    "us-west-2a", 
    "us-west-2b", 
    "us-west-2c", 
    "us-west-2d"
]

http://ec2-35-89-224-234.us-west-2.compute.amazonaws.com/cafe
Number of orders: 45

aws ec2 create-security-group \
--group-name CafeDatabaseSG \
--description "Security group for Cafe database" \
--vpc-id vpc-0fb54f9536bfbaf86
{
    "GroupId": "sg-0602bdef54dc7032e"
}

aws ec2 authorize-security-group-ingress \
--group-id sg-0602bdef54dc7032e \
--protocol tcp --port 3306 \
--source-group sg-0bc4f0499b69584ea

aws ec2 describe-security-groups \
--query "SecurityGroups[*].[GroupName,GroupId,IpPermissions]" \
--filters "Name=group-name,Values='CafeDatabaseSG'"
[
    [
        "CafeDatabaseSG", 
        "sg-0602bdef54dc7032e", 
        [
            {
                "PrefixListIds": [], 
                "FromPort": 3306, 
                "IpRanges": [], 
                "ToPort": 3306, 
                "IpProtocol": "tcp", 
                "UserIdGroupPairs": [
                    {
                        "UserId": "332064123035", 
                        "GroupId": "sg-0bc4f0499b69584ea"
                    }
                ], 
                "Ipv6Ranges": []
            }
        ]
    ]
]

aws ec2 create-subnet \
--vpc-id vpc-0fb54f9536bfbaf86 \
--cidr-block 10.200.2.0/23 \
--availability-zone us-west-2a
{
    "Subnet": {
        "MapPublicIpOnLaunch": false, 
        "AvailabilityZoneId": "usw2-az2", 
        "AvailableIpAddressCount": 507, 
        "DefaultForAz": false, 
        "SubnetArn": "arn:aws:ec2:us-west-2:332064123035:subnet/subnet-00fc48ba220fa60f7", 
        "Ipv6CidrBlockAssociationSet": [], 
        "VpcId": "vpc-0fb54f9536bfbaf86", 
        "State": "available", 
        "AvailabilityZone": "us-west-2a", 
        "SubnetId": "subnet-00fc48ba220fa60f7", 
        "OwnerId": "332064123035", 
        "CidrBlock": "10.200.2.0/23", 
        "AssignIpv6AddressOnCreation": false
    }
}

Cafe Private Subnet 1 ID: subnet-00fc48ba220fa60f7

aws ec2 create-subnet \
--vpc-id vpc-0fb54f9536bfbaf86 \
--cidr-block 10.200.5.0/23 \
--availability-zone us-west-2b
{
    "Subnet": {
        "MapPublicIpOnLaunch": false, 
        "AvailabilityZoneId": "usw2-az1", 
        "AvailableIpAddressCount": 507, 
        "DefaultForAz": false, 
        "SubnetArn": "arn:aws:ec2:us-west-2:332064123035:subnet/subnet-0cdf21a576baa93e2", 
        "Ipv6CidrBlockAssociationSet": [], 
        "VpcId": "vpc-0fb54f9536bfbaf86", 
        "State": "available", 
        "AvailabilityZone": "us-west-2b", 
        "SubnetId": "subnet-0cdf21a576baa93e2", 
        "OwnerId": "332064123035", 
        "CidrBlock": "10.200.4.0/23", 
        "AssignIpv6AddressOnCreation": false
    }
}
Cafe Private Subnet 2 ID: subnet-0cdf21a576baa93e2

aws rds create-db-subnet-group \
--db-subnet-group-name "CafeDB Subnet Group" \
--db-subnet-group-description "DB subnet group for Cafe" \
--subnet-ids subnet-00fc48ba220fa60f7 subnet-0cdf21a576baa93e2 \
--tags "Key=Name,Value= CafeDatabaseSubnetGroup"
{
    "DBSubnetGroup": {
        "Subnets": [
            {
                "SubnetStatus": "Active", 
                "SubnetIdentifier": "subnet-0cdf21a576baa93e2", 
                "SubnetOutpost": {}, 
                "SubnetAvailabilityZone": {
                    "Name": "us-west-2b"
                }
            }, 
            {
                "SubnetStatus": "Active", 
                "SubnetIdentifier": "subnet-00fc48ba220fa60f7", 
                "SubnetOutpost": {}, 
                "SubnetAvailabilityZone": {
                    "Name": "us-west-2a"
                }
            }
        ], 
        "VpcId": "vpc-0fb54f9536bfbaf86", 
        "DBSubnetGroupDescription": "DB subnet group for Cafe", 
        "SubnetGroupStatus": "Complete", 
        "DBSubnetGroupArn": "arn:aws:rds:us-west-2:332064123035:subgrp:cafedb subnet group", 
        "DBSubnetGroupName": "cafedb subnet group"
    }
}

From exercise:
    DB instance identifier: CafeDBInstance
    Engine option: MariaDB
    DB engine version: 10.5.13
    DB instance class: db.t3.micro
    Allocated storage: 20 GB
    Availability Zone: CafeInstance Availability Zone
    DB Subnet group: CafeDB Subnet Group
    VPC security groups: CafeDatabaseSG
    Public accessibility: No
    Username: root
    Password: Re:Start!9

    CafeInstance Instance ID: i-02347e69026d68b65
    CafeInstance Instance Type: t3.small
    CafeInstance Public DNS Name: ec2-35-89-224-234.us-west-2.compute.amazonaws.com
    CafeInstance Public IP Address: 35.89.224.234
    CafeInstance Availability Zone: us-west-2a
    CafeInstance VPC ID: vpc-0fb54f9536bfbaf86
    CafeSecurityGroup Group ID: sg-0bc4f0499b69584ea

aws rds create-db-instance \
--db-instance-identifier CafeDBInstance \
--engine mariadb \
--engine-version 10.5.13 \
--db-instance-class db.t3.micro \
--allocated-storage 20 \
--availability-zone us-west-2a \
--db-subnet-group-name "CafeDB Subnet Group" \
--vpc-security-group-ids sg-0602bdef54dc7032e \
--no-publicly-accessible \
--master-username root --master-user-password 'Re:Start!9'
{
    "DBInstance": {
        "PubliclyAccessible": false, 
        "MasterUsername": "root", 
        "MonitoringInterval": 0, 
        "LicenseModel": "general-public-license", 
        "VpcSecurityGroups": [
            {
                "Status": "active", 
                "VpcSecurityGroupId": "sg-0602bdef54dc7032e"
            }
        ], 
        "CopyTagsToSnapshot": false, 
        "OptionGroupMemberships": [
            {
                "Status": "in-sync", 
                "OptionGroupName": "default:mariadb-10-5"
            }
        ], 
        "PendingModifiedValues": {
            "MasterUserPassword": "****"
        }, 
        "Engine": "mariadb", 
        "MultiAZ": false, 
        "DBSecurityGroups": [], 
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mariadb10.5", 
                "ParameterApplyStatus": "in-sync"
            }
        ], 
        "PerformanceInsightsEnabled": false, 
        "AutoMinorVersionUpgrade": true, 
        "PreferredBackupWindow": "13:16-13:46", 
        "DBSubnetGroup": {
            "Subnets": [
                {
                    "SubnetStatus": "Active", 
                    "SubnetIdentifier": "subnet-0cdf21a576baa93e2", 
                    "SubnetOutpost": {}, 
                    "SubnetAvailabilityZone": {
                        "Name": "us-west-2b"
                    }
                }, 
                {
                    "SubnetStatus": "Active", 
                    "SubnetIdentifier": "subnet-00fc48ba220fa60f7", 
                    "SubnetOutpost": {}, 
                    "SubnetAvailabilityZone": {
                        "Name": "us-west-2a"
                    }
                }
            ], 
            "DBSubnetGroupName": "cafedb subnet group", 
            "VpcId": "vpc-0fb54f9536bfbaf86", 
            "DBSubnetGroupDescription": "DB subnet group for Cafe", 
            "SubnetGroupStatus": "Complete"
        }, 
        "ReadReplicaDBInstanceIdentifiers": [], 
        "AllocatedStorage": 20, 
        "DBInstanceArn": "arn:aws:rds:us-west-2:332064123035:db:cafedbinstance", 
        "BackupRetentionPeriod": 1, 
        "PreferredMaintenanceWindow": "sat:07:23-sat:07:53", 
        "DBInstanceStatus": "creating", 
        "IAMDatabaseAuthenticationEnabled": false, 
        "EngineVersion": "10.5.13", 
        "DeletionProtection": false, 
        "AvailabilityZone": "us-west-2a", 
        "DomainMemberships": [], 
        "StorageType": "gp2", 
        "DbiResourceId": "db-M6H3FMMFOEODOSWDHDAKBMULSM", 
        "CACertificateIdentifier": "rds-ca-2019", 
        "StorageEncrypted": false, 
        "AssociatedRoles": [], 
        "DBInstanceClass": "db.t3.micro", 
        "DbInstancePort": 0, 
        "DBInstanceIdentifier": "cafedbinstance"
    }
}

aws rds describe-db-instances \
--db-instance-identifier CafeDBInstance \
--query "DBInstances[*].[Endpoint.Address,AvailabilityZone,PreferredBackupWindow,BackupRetentionPeriod,DBInstanceStatus]"
[
    [
        null, 
        "us-west-2a", 
        "13:16-13:46", 
        1, 
        "creating"
    ]
]

RDS Instance Database Endpoint Address: cafedbinstance.cj5a3tqv94tb.us-west-2.rds.amazonaws.com

CafeInstance Public IP Address: 35.89.224.234

mysqldump --user=root --password='Re:Start!9' \
--databases cafe_db --add-drop-database > cafedb-backup.sql


mysql --user=root --password='Re:Start!9' \
--host=cafedbinstance.cj5a3tqv94tb.us-west-2.rds.amazonaws.com \
< cafedb-backup.sql

mysql --user=root --password='Re:Start!9' \
--host=cafedbinstance.cj5a3tqv94tb.us-west-2.rds.amazonaws.com \
cafe_db
"Welcome to the MariaDB monitor. ..."

http://ec2-35-89-224-234.us-west-2.compute.amazonaws.com/cafe

mysql --user=root --password='Re:Start!9' \
--host=cafedbinstance.cj5a3tqv94tb.us-west-2.rds.amazonaws.com \
cafe_db
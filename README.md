AWS Network+Architecture
========================

This repository contains an advanced CloudFormation template (`AWS Network+Architecture`) that provisions a multi-AZ VPC, Application Load Balancer (ALB), AutoScaling Group (ASG), NAT Gateways, SSM-capable instances (SSM Instance Profile), and secure defaults (IMDSv2, CreationPolicy/cfn-signal integration).

Quick summary
- Template path: `AWS Network+Architecture` (root)
- Purpose: demo/prod-ready network + app stack for AWS using CloudFormation
- Key features: multi-AZ public/private subnets, NAT per AZ, ALB, ASG, SSM instance profile, IMDSv2 enforced, EIPs with DeletionPolicy Retain.

Optional extended description
---------------------------
You can place an extended description here describing the repository intent, ownership, deployment process, environment naming, and any compliance or cost considerations. Example:

"This stack is intended for sandbox and development use. For production use, adjust CIDRs, enable monitoring and alarms (CloudWatch), use Launch Templates, add AutoScaling lifecycle hooks, and store state and templates in an IaC pipeline (CodePipeline/CodeBuild, Terraform Cloud, etc.). Avoid opening broad CIDR ranges for SSH and HTTP on security groups; prefer ALB for public traffic and Session Manager for operations. NAT Gateways incur hourly and data charges; plan accordingly."

How to validate locally
-----------------------
1. Install and configure AWS CLI and authenticate (`aws configure`) or use a named profile.
2. Run CloudFormation validate:

```powershell
aws cloudformation validate-template --template-body file://"c:\Users\################################################################
```

How to push to GitHub
---------------------
If you have the GitHub CLI (`gh`) installed and authenticated:

```powershell
cd "c:\Users\###########\OneDrive\Documents\AWS-Cloud+Security"
gh repo create MegaHax/aws-network-architecture --public --source=. --remote=origin --push
```

Or create a repo on github.com and then:

```powershell
cd "c:\Users\############\OneDrive\Documents\AWS-Cloud+Security"
git remote add origin [https://github.com/Megahax0x/AWS-Network-Archirecture-aws-cli-demo
git push -u origin main
```

Contact / Ownership
-------------------
Owner: MegaHax <@Megahax0x>


(You can edit this README to add a longer project description, contribution guidelines, and deployment steps.)

# AWS-Network-Archirecture-aws-cli-demo
Demo project for configuring AWS CLI
AWSTemplateFormatVersion: '2025-05-10'
Description: Simple AWS network + bastion + app instance CloudFormation template for Console

Parameters:
    KeyName:
        Description: EC2 KeyPair name for SSH access to bastion (required)
        Type: AWS::EC2::KeyPair::KeyName
    MyIp:
        Description: Your IPv4 CIDR for SSH access to bastion (e.g. 203.0.113.0/32)
        Type: String
        Default: 0.0.0.0/0
    LatestAmi:
        Description: Amazon Linux 2 AMI (SSM parameter)
        Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
        Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
    VPC:
        Type: AWS::EC2::VPC
        Properties:
            CidrBlock: 10.0.0.0/16
            EnableDnsSupport: true
            EnableDnsHostnames: true
            Tags:
                - Key: Name
                    Value: simple-vpc

    PublicSubnet:
        Type: AWS::EC2::Subnet
        Properties:
            VpcId: !Ref VPC
            CidrBlock: 10.0.1.0/24
            MapPublicIpOnLaunch: true
            Tags:
                - Key: Name
                    Value: public-subnet

    PrivateSubnet:
        Type: AWS::EC2::Subnet
        Properties:
            VpcId: !Ref VPC
            CidrBlock: 10.0.2.0/24
            MapPublicIpOnLaunch: false
            Tags:
                - Key: Name
                    Value: private-subnet

    InternetGateway:
        Type: AWS::EC2::InternetGateway
        Properties:
            Tags:
                - Key: Name
                    Value: igw

    VPCGatewayAttachment:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
            VpcId: !Ref VPC
            InternetGatewayId: !Ref InternetGateway

    PublicRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
            VpcId: !Ref VPC
            Tags:
                - Key: Name
                    Value: public-rt

    PublicDefaultRoute:
        Type: AWS::EC2::Route
        DependsOn: VPCGatewayAttachment
        Properties:
            RouteTableId: !Ref PublicRouteTable
            DestinationCidrBlock: 0.0.0.0/0
            GatewayId: !Ref InternetGateway

    PublicSubnetRouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
            SubnetId: !Ref PublicSubnet
            RouteTableId: !Ref PublicRouteTable

    EIPForNAT:
        Type: AWS::EC2::EIP
        Properties:
            Domain: vpc

    NATGateway:
        Type: AWS::EC2::NatGateway
        Properties:
            AllocationId: !GetAtt EIPForNAT.AllocationId
            SubnetId: !Ref PublicSubnet
            Tags:
                - Key: Name
                    Value: nat-gw

    PrivateRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
            VpcId: !Ref VPC
            Tags:
                - Key: Name
                    Value: private-rt

    PrivateDefaultRoute:
        Type: AWS::EC2::Route
        Properties:
            RouteTableId: !Ref PrivateRouteTable
            DestinationCidrBlock: 0.0.0.0/0
            NatGatewayId: !Ref NATGateway

    PrivateSubnetRouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
            SubnetId: !Ref PrivateSubnet
            RouteTableId: !Ref PrivateRouteTable

    BastionSG:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: Bastion SG: allow SSH from MyIp and HTTP
            VpcId: !Ref VPC
            SecurityGroupIngress:
                - IpProtocol: tcp
                    FromPort: 22
                    ToPort: 22
                    CidrIp: !Ref MyIp
                - IpProtocol: tcp
                    FromPort: 80
                    ToPort: 80
                    CidrIp: 0.0.0.0/0
            Tags:
                - Key: Name
                    Value: bastion-sg

    AppSG:
        Type: AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: App SG: allow HTTP from anywhere, SSH from bastion
            VpcId: !Ref VPC
            SecurityGroupIngress:
                - IpProtocol: tcp
                    FromPort: 80
                    ToPort: 80
                    CidrIp: 0.0.0.0/0
                - IpProtocol: tcp
                    FromPort: 22
                    ToPort: 22
                    SourceSecurityGroupId: !Ref BastionSG
            Tags:
                - Key: Name
                    Value: app-sg

    BastionInstance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t3.micro
            KeyName: !Ref KeyName
            ImageId: !Ref LatestAmi
            NetworkInterfaces:
                - AssociatePublicIpAddress: true
                    DeviceIndex: 0
                    SubnetId: !Ref PublicSubnet
                    GroupSet:
                        - !Ref BastionSG
            Tags:
                - Key: Name
                    Value: bastion

    AppInstance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t3.micro
            KeyName: !Ref KeyName
            ImageId: !Ref LatestAmi
            SubnetId: !Ref PrivateSubnet
            SecurityGroupIds:
                - !Ref AppSG
            Tags:
                - Key: Name
                    Value: app-instance
            UserData:
                Fn::Base64: |
                    #!/bin/bash
                    yum update -y
                    yum install -y httpd
                    systemctl enable httpd
                    systemctl start httpd
                    echo "Hello from private app instance" > /var/www/html/index.html

Outputs:
    VPCId:
        Description: VPC ID
        Value: !Ref VPC
    BastionPublicIP:
        Description: Public IP of Bastion
        Value: !GetAtt BastionInstance.PublicIp
    AppPrivateIP:
        Description: Private IP of App Instance
        Value: !GetAtt AppInstance.PrivateIp

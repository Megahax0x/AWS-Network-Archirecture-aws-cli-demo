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





This stack is designed for development and sandbox environments. For production use, it is recommended to adjust CIDR ranges, enable monitoring and alarms (CloudWatch), use Launch Templates, add AutoScaling lifecycle hooks, and store state and templates in an infrastructure-as-code pipeline (e.g., CodePipeline/CodeBuild, Terraform Cloud).

Avoid opening broad CIDR ranges for SSH and HTTP in security groups; prefer using the ALB for public traffic and Session Manager for administrative operations. NAT Gateways incur hourly and data transfer charges—plan their usage carefully.


Contact / Ownership
-------------------
Owner: MegaHax <@Megahax0x>


(You can edit this README to add a longer project description, contribution guidelines, and deployment steps.)

# AWS-Network-Archirecture-aws-cli-demo
Demo project for configuring AWS CLI
AWSTemplateFormatVersion: '2025-05-10'
Description: Simple AWS network + bastion + app instance CloudFormation template for Console


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

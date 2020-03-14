# Weatherapp -  Ansible Deploy to AWS
---

This is the Ansible section complementing the [Weatherapp][1] challenge/project.

## Prerequisites

* From AWS IAM, create a new user named `ansible` and give it programmatic access ONLY
* Copy the newly generated access key ID and secret access key before moving on
* Create a git-ignored file called `aws_secrets` in the root directory with the following information:
    ```bash
    export AWS_ACCESS_KEY_ID="<YOUR_ANSIBLE_ACCESS_KEY_ID>"
    export AWS_SECRET_ACCESS_KEY="<YOUR_ANSIBLE_SECRET_ACCESS_KEY>"
    export AWS_REGION="<YOUR_DESIRED_REGION>"
* Attach the `AdministratorAccess` AWS-managed policy to `ansible` user
    * _This will be changed later to only allow access as necessary_

## Ansible Steps

* Locally source `ansible` user access information
* Ensure presence of default VPC in the region
* Gather default VPC and subnets facts
* Create a new SSH keypair
* Copy keypair to `.keypairs` folder with the correct file mode
* Ensure that `.keypairs` folder is git-ignored
* Create a new security group with limited SSH access and public HTTP
* Create an EC2 instance within the security group and gather facts
* Add instance to inventory
* Install Docker and essential packages
* Transfer Docker images to EC2 (If pulled from hub, use docker login with vaulted file)
* Create an Amazon Machine Image (AMI) from the instance
* Create an Elastic Load Balancer (ELB) and apply health checks
* Gather ELB facts and note the domain name and IP
* Create a launch configuration/template with AMI
* Create an auto-scaling group using the launch configuration and ELB health checks

___Note to self:__ Make sure to use resource tagging, and have a reversal/destroy option_

___
___Note:__ Terraform or CloudFormation are more "right" Infrastructure as Code (IaC) tools than Ansible
(especially for more complicated setups) but this is an Ansible exercise in trying out different modules.
I may actually revisit this project later to use one of them for the infrastructure building phases._
___


[1]: https://github.com/aihaddad/weatherapp
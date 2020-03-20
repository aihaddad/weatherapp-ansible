# Weatherapp -  Ansible Deploy to AWS
---

This is the Ansible section complementing the [Weatherapp][1] challenge/project.

## Prerequisites

* Clone the repo and `cd weatherapp-ansible`
* From AWS IAM, create a new user named `ansible` and give it programmatic access ONLY
* Copy the newly generated access key ID and secret access key before moving on
* Create a git-ignored file called `aws_secrets` in the root directory with the following information:
    ```bash
    export AWS_ACCESS_KEY_ID="<YOUR_ANSIBLE_ACCESS_KEY_ID>"
    export AWS_SECRET_ACCESS_KEY="<YOUR_ANSIBLE_SECRET_ACCESS_KEY>"
    export AWS_REGION="<YOUR_DESIRED_REGION>"
* You can skip this step if you have `aws-cli` configured with the same information
* Attach the `AdministratorAccess` AWS-managed policy to `ansible` user
    * _This may be changed later to only allow access as necessary_
* Make sure to have a default VPC built for the AWS region specified

## Ansible Steps

* Locally source `ansible` user access information
* Gather __default__ VPC and subnets facts
* Create a new SSH key pair
* Copy key pair to `.keypairs` folder with the correct file mode
* Ensure that `.keypairs` folder is git-ignored
* Create a new security group with limited SSH access and public HTTP
* Create an EC2 instance within the security group and gather facts
* Add instance to inventory
* Install Docker and essential packages
* Deploy by pulling from GitHub and using docker-compose
* Create an Amazon Machine Image (AMI) from the instance
* Create an Elastic Load Balancer (ELB) and apply health checks
* Gather ELB facts and note the domain name and IP
* _(Optional)_ Create a launch configuration/template with AMI
* _(Optional)_ Create an auto-scaling group using the launch configuration and ELB health checks

___
___Note:__ Terraform or CloudFormation are more "right" Infrastructure as Code (IaC) tools than Ansible
(especially for more complicated setups and managing rollbacks) but this is an Ansible exercise in trying out different modules.
I may  revisit this project later to use one of those tools for a more complete infrastructure build._
___

## Usage

### Deploy a base webserver


* Once ready to deploy, load the AWS credentials _(skip if using `aws-cli`)_:
    ```bash
    $ source ./aws_secrets
    ```
* You can add it with absolute path to your `.bash_profile`
* Check the path to your system or venv Python interpreter and update the inventory file
* Make sure the AWS Boto3 and Botocore packages are installed on that Python
* Update the `./group_vars/webservers/vars` file to point to your Docker-ized app repo
* Also, make sure to make the necessary edits with the backend service URL and port
* In the same folder, create a new file `secrets` and set your OpenWeatherMap key as follows;
    ```yml
    vault_app_id: <YOUR_OPENWEATHERMAP_KEY>
    ```
* The file will be git-ignored, but you can use `ansible-vault` to encrypt it for extra security
* You now have access to three deployment playbooks in the folder root
    1. Run `$ ansible-playbook base.yml` to:
        * Gather information for a subnet in the default VPC
        * Get the ID of the latest Amazon Linux 2 AMI
        * Create a new SSH key pair, and webserver security group
        * Build the base webserver EC2 instance using all previous details
            * _The VM type is t3.nano, but you can change it in `./group_vars/all/vars`_
        * Add the created instance to the inventory file
    2. Run `$ ansible-playbook setup.yml` to:
        * Update the Linux system packages
        * Install Git, Docker and docker-compose
        * Start the Docker service
    3. Run `ansible-playbook deploy.yml` to:
        * Pull the app from the repo
        * Set the proper environment variables
        * Cleanup previous Docker containers and images
        * Build and run the app using docker-compose
* You can now visit the instance public IP or DNS to see the running app
* If you have the `secrets` variables file encrypted, make sure to use the `--ask-vault-pass` or `--vault-id xxx@prompt` flags in steps `ii` and `iii`
* Alternatively, and highly not recommended, you can deploy using the inefficient `deploy_tar.yml` playbook method to upload the docker images pre-built locally. Just make sure they have the production environment variables set.
* To rollback/destroy all the base AWS resources created, run:
    * `$ ansible-playbook base.yml --tags=cleanup` 


[1]: https://github.com/aihaddad/weatherapp
# Weatherapp -  Ansible Deploy to AWS
---

This is the Ansible section complementing the [Weatherapp][1] challenge/project.
This solution includes playbooks for the creation of a Base web server instance, Docker setup, AMI creation, scaling out and in, load balancing and deployment with two methods: an inefficient one with pre-built image tarballs, and an efficient one using git and docker-compose.
It also includes tasks for rolling-back/destroying the resources it created.

## Prerequisites

* Clone the repo and `cd weatherapp-ansible`
* From AWS IAM, create a new user named `ansible` and give it programmatic access ONLY
* Copy the newly generated access key ID and secret access key before moving on
* Create a git-ignored file called `aws_secrets` in the root directory with the following information:
    ```bash
    export AWS_ACCESS_KEY_ID="<YOUR_ANSIBLE_ACCESS_KEY_ID>"
    export AWS_SECRET_ACCESS_KEY="<YOUR_ANSIBLE_SECRET_ACCESS_KEY>"
    export AWS_REGION="<YOUR_DESIRED_REGION>"
    ```
* You can skip this step if you have `aws-cli` configured with the same information
* Attach the `AdministratorAccess` AWS-managed policy to `ansible` user
    * _This may be changed later to only allow access as necessary_
* Make sure to have a default VPC built for the AWS region specified

## Ansible Steps

* Locally source `ansible` user access information
* Gather __default__ VPC and subnets facts
* Create a new SSH key pair and copy to a `.keypairs` folder with the correct file mode
* Ensure that `.keypairs` folder is git-ignored
* Create a new security group with limited SSH access and public HTTP
* Create an EC2 instance within the security group and gather facts
* Add instance to inventory
* Install Docker and essential packages
* Deploy by pulling from GitHub and using docker-compose
* Create an Amazon Machine Image (AMI) from the instance
* Create a scaling workflow
* Create an Elastic Load Balancer (ELB) with basic health checks and note the domain name

___
___Note:__ Terraform or CloudFormation are more "right" Infrastructure as Code (IaC) tools than Ansible
(especially for more complicated setups and managing rollbacks) but this is an Ansible exercise in trying out different modules.
I may  revisit this project later to use one of those tools for a more complete infrastructure build._
___

## Usage

### Deploy a base webserver

* Once the app is ready to deploy, load the AWS credentials _(Skip if it's not needed)_:
    ```bash
    $ source ./aws_secrets
    ```
* You can add it with absolute path to your `.bash_profile`
* Check the path to your system or venv Python interpreter and update the `ansible_python_interpreter` field in the inventory file
* Make sure the AWS Boto3 and Botocore packages are installed on that Python
* Update the `./group_vars/webservers/vars` file to point to your Docker-ized app repo
* Also, make sure to make the necessary edits with the backend service URL and port if any are required
* In the same folder, create a new file `secrets` and set your OpenWeatherMap key as follows;
    ```yml
    vault_app_id: <YOUR_OPENWEATHERMAP_KEY>
    ```
* The file will be git-ignored, but you can use `ansible-vault` to encrypt it for extra security
* You now have access to three deployment playbooks in the folder root
    1. Run `$ ansible-playbook base.yml` to:
        * Gather information for a subnet in the default VPC
        * Get the ID of the latest Amazon Linux 2 AMI
        * Create a "facts" folder to hold the information in YAML
        * Create a new SSH key pair, and webserver security group
        * Build the base webserver EC2 instance using all previous details
            * _The VM type is t3.nano, but you can change it in `./group_vars/all/vars`_
        * Add the created instance to the inventory file
        * Create an ELB and target group to be used later when scaling
        * Note the DNS name of the ELB to the file `facts/elb_url.yml`
    2. Run `$ ansible-playbook setup.yml` to:
        * Update the Linux system packages
        * Install Git, Docker and docker-compose
        * Start and enable the Docker service
        * Reboot the EC2 instance
        * Create an AMI of the instance
    3. Run `ansible-playbook deploy.yml` to:
        * Pull the app from the repo
        * Set the proper environment variables
        * Cleanup previous Docker containers and images
        * Build and run the app using docker-compose
        * You can now see the app running on its public DNS/IP or the URL to the ELB
    5. [`as needed`] Run `ansible-playbook ami.yml` to:
        * Create AMIs manually for your RPO needs
    4. [`as needed`] Run `ansible-playbook scale.yml` to:
        * Scale out/add 3 new webserver instances based on the created AMI
            * _3 is the default based on the number of AZ's in the EU-North-1 region_
            * _This `scale_count` variable can be changed in the `group_vars/all/vars` file_
        * Add the created instances to the inventory
    5. [`as needed`] Run `ansible-playbook scale.yml --tags in` to:
        * Scale in/terminate 1 webserver instance and remove it from the target group
        * Remove its reference in the inventory

* Whenever you run the `deploy.yml` later, it will deploy to all the webservers
* If you have the `secrets` variables file encrypted, make sure to use the `--ask-vault-pass` or `--vault-id xxx@prompt` flags in steps `ii` and `iii`
* Alternatively, and __highly not recommended__, you can deploy using the inefficient `deploy_tar.yml` playbook method to upload the docker images pre-built locally. Just make sure they have the production environment variables set.
* To rollback/destroy all the AWS resources created, run:
    * `$ ansible-playbook base.yml --tags=cleanup`
    * `$ ansible-playbook ami.yml --tags=cleanup`


[1]: https://github.com/aihaddad/weatherapp
# Launch a VM and install wordpress on it using terraform and ansible
# Introduction
This guide aims to walk you through the process of launching a virtual machine (VM) and installing WordPress on Amazon Web Services (AWS) using Terraform for infrastructure provisioning and Ansible for configuration management

# Why Terraform and Ansible?
- __Terraform:__ Infrastructure as Code (IaC) tools like Terraform allow you to define and provision infrastructure in a declarative manner. With Terraform, you can easily create and manage AWS resources, making infrastructure deployment predictable and scalable.
- __Ansible:__ Configuration management tools like Ansible automate the process of configuring and managing servers. Ansible enables you to define the desired state of your servers, ensuring consistency and repeatability in your deployments.

# Prerequisites
Before you begin, make sure you have the following prerequisites in place:

1. __Terraform Installation:__
Follow the official [Terraform installation guide](https://developer.hashicorp.com/terraform/install) to install Terraform on your machine.
2. __Ansible Installation:__
Follow the official [Ansible installation guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible.
3. __AWS Account:__
Create an AWS account if you don't have one already. This guide assumes you have AWS credentials configured on your machine. I will discuss how to provide credentials later.

# Overview of the Process:
1. __Terraform Configuration (Infrastructure Provisioning):__
Use Terraform to define the AWS resources needed, such as EC2 instances, security groups, and key pairs.
2. __Terraform Execution:__
Initialize the Terraform project and apply the configuration to create the specified AWS resources.
3. __Ansible Playbook (Configuration Management):__
Create an Ansible playbook to configure the EC2 instance, install necessary software (Apache, MySQL, PHP, and WordPress), and set up WordPress.
4. __Run Ansible Playbook:__
Execute the Ansible playbook to apply the configuration to the launched EC2 instance.
5. __Access WordPress:__
Open a web browser and access the WordPress site on the public IP of the EC2 instance.
6. ___Optional: Resource Cleanup:___
If desired, use Terraform to destroy the AWS resources created during the process.

# Let's Get Started:
Proceed to the next sections to follow the step-by-step guide on launching a VM and installing WordPress using Terraform and Ansible.

## Step 1: Create Terraform Configuration
### Create files and directories
Create a directory structure for this project, you can use next script to create the structure.
```bash
# Create directories
mkdir -p .ssh modules/ec2

# Create Terraform files
touch main.tf outputs.tf providers.tf variables.tf
touch modules/ec2/main.tf modules/ec2/outputs.tf modules/ec2/variables.tf
```
After execution you'll get next structure
```
📦 Project Directory
 ┣ 📂.ssh
 ┣ 📂modules
 ┃ ┗ 📂ec2
 ┃ ┃ ┣ 📜main.tf
 ┃ ┃ ┣ 📜outputs.tf
 ┃ ┃ ┗ 📜variables.tf
 ┣ 📜main.tf
 ┣ 📜outputs.tf
 ┣ 📜providers.tf
 ┗ 📜variables.tf
```
This directory structure is organized for a Terraform project. Let's break down the contents:

- __.ssh:__ A directory for storing SSH-related files, presumably for key pairs or other SSH configurations.
- __modules:__ This directory contains reusable Terraform modules that can be used in the main configuration.
  - __ec2:__ Another Terraform module for EC2-related configurations. Similar to the Docker module, it includes the main configuration (main.tf), outputs (outputs.tf), and variables (variables.tf).
- __main.tf:__ The main Terraform configuration file for the project.
- __outputs.tf:__ A file where you can define the output values of your Terraform configurations.
- __providers.tf:__ This file typically contains provider-specific configurations (e.g., AWS, Docker) for the main Terraform configuration.
- __variables.tf:__ A file for declaring input variables used in your Terraform configurations.

This structure follows good Terraform practices by organizing configurations into modules, separating concerns, and providing clear entry points for the main configuration and modules.

### Configure providers
Inside you directory open `main.tf` file and add AWS and Docker providers.
```bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

As I mentioned before, here I will discuss AWS credentials. But firstly, you need create access keys for you AWS account, to do this read [this guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). Here are two methods to use AWS credentials:
1. Create environment variables on your machine 
```bash
export AWS_ACCESS_KEY_ID="<your access key>"
export AWS_SECRET_ACCESS_KEY="<your secret key>"
export AWS_DEFAULT_REGION="<region>"
```
2. Create AWS configuration files. Do do this you need to create a directory in the root of this project, or in more secure place on your machine.
```bash
📦 Project Directory
 ┣ 📂.aws
 ┃ ┣ 📜conf
 ┗ ┗ 📜creds
```
- __conf__: This file typically stores configuration settings for the AWS CLI (Command Line Interface). It can include options such as the default region, output format, and other CLI-related configurations.
```ini
[default]
region = us-east-1
output = json
```
- __credentials:__ This file is used to store AWS access key ID and secret access key. These credentials are necessary for authenticating your AWS CLI commands and API requests.
```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY
```
After this you need to specify paths to this files in the `providers.tf`, so terraform can read them and access AWS.
```bash
provider "aws" {
  shared_config_files      = [".aws/config"]
  shared_credentials_files = [".aws/credentials"]
  profile                  = "default"
}
```

### Create SSH keys
Run the following command to generate a new SSH key pair. 
```bash
ssh-keygen -t rsa -b 4096
```
This command will generate a new RSA key pair with a bit length of 4096.

You will be prompted to choose a location to save the new key. Enter full path fo `.ssh` that we created and add id_rsa in the end (e.g /home/user/wordpress_tf/.ssh/id_rsa). This will save keys to our directory.

You can also choose to set a passphrase for additional security. If you don't want to set a passphrase, simply press Enter twice.

After completing these steps, the command will generate two files: /home/user/wordpress_tf/.ssh/id_rsa (private key) and /home/user/wordpress_tf/.ssh/id_rsa (public key).

## Step2: Create AWS module
The ASW module may define resources suach as AWS EC2, Security Group and Key-Pair. This is all that we need to use in case of this project.

### `aws_security_group`
This resource defines an AWS Security Group, which acts as a virtual firewall for your instance. It allows or denies inbound and outbound traffic based on specified rules. The defined ingress rules allow traffic on ports 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL), and 5666 (NRPE). Optionally, it allows ICMP traffic if the icmp variable is set to true.

```hcl
# modules/ec2/main.tf
resource "aws_security_group" "this" {
  name = var.sg_name

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 5666
    to_port     = 5666
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  dynamic "ingress" {
    for_each = var.icmp ? [1] : []
    content {
      from_port   = -1
      to_port     = -1
      protocol    = "icmp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
### `aws_key_pair`
This resource creates an AWS key pair used for SSH access to the EC2 instance. It takes the public key from the specified file (var.key_path) and associates it with the specified key_name.

```hcl
# modules/ec2/main.tf
resource "aws_key_pair" "this" {
  key_name   = var.key_name
  public_key = file(var.key_path)
}
```
### `aws_instance`
This resource defines the EC2 instance itself. Key attributes include:

- __ami:__ The Amazon Machine Image (AMI) ID to launch the instance.
- __instance_type:__ The type of instance (t2.micro, t3.micro is aliable to Free Tier).
- __security_groups:__ A list of security group names to associate with the instance.
- __key_name:__ The name of the key pair to use for the instance.
- __root_block_device:__ Specifies the root block device settings, in this case, a volume size of 20 GB.
- __tags:__ Tags to apply to the EC2 instance. In this case it is the name of the instance.

```hcl
# modules/ec2/main.tf
resource "aws_instance" "this" {
  ami             = var.ami
  instance_type   = var.instance_type
  security_groups = [aws_security_group.this.name]

  key_name = aws_key_pair.this.key_name

  root_block_device {
    volume_size = 20
  }

  tags = {
    Name = var.instance_name
  }
}
```

### Create Variables
In the `variables.tf` file you need to specify all variables that used in EC2 configuration. This is variables that terraform reads from user. It can be set by console or bu `vars.tfvars` file in the root directory. In modules this file used to assign variables when creates modules in root file. You can set default value, description and type of each variable.

```hcl
# module/ec2/variables.tf
variable "key_name" {
  default     = "wordpress_tf"
  type        = string
  description = "The name of the SSH key to use"
}

variable "key_path" {
  default     = "~/.ssh/id_rsa.pub"
  type        = string
  description = "The path to the SSH key to use"
}

variable "instance_name" {
  default     = "wordpress-instance-1"
  type        = string
  description = "The name of the EC2 instance"
}

variable "ami" {
  default     = "ami-xxxxxxxxxxxxxxxxxx"
  type        = string
  description = "The AMI to use for the EC2 instance"
}
variable "instance_type" {
  default     = "t2.micro"
  type        = string
  description = "The type of EC2 instance to use"
}

variable "sg_name" {
  default     = "wordpress_sg"
  type        = string
  description = "The name of the security group"
}

variable "icmp" {
  default = false
  type    = bool
  description = "Whether to allow ICMP traffic"
}
```

### Create outputs
The `outputs.tf` file in a module is used to declare values that the module will output. These outputs can then be referenced by the calling or parent module.
```hcl
# modules/ec2/outputs.tf
output "public_ip" {
  value = aws_instance.wordpress.public_ip
}
```
In this case, we need only Public IP, so we can visit Wordpress site in the future.

## Step 3: Create module in root
The last one step before applying terraform is to define ec2 module in the root files, so we can use it. Firstly, lets specify module in `main.tf`.

```hcl
# main.tf
module "wordpress_ec2" {
  source = "./modules/ec2"

  instance_name = "wordpress-instance-1"
  ami           = var.image
  instance_type = "t2.micro"

  sg_name  = "wordpress-sg"
  icmp     = true
  key_name = "wordpress"
  key_path = "${path.module}/.ssh/id_rsa.pub"
}
```

I used here `${path.module}` expression here. It defines the path to module where it runs, in this case root directory. We need this to specify SSH keys correctly.

### Define Variables
In this case I only specified AMI image variable for example of how it works. You can specify other variables if you need.
```hcl
variable "image" {
  description = "The AMI image for EC2 instance"
  default     = "ami-xxxxxxxxxxxxxxx"
  type        = string
}
```

### Define Outputs
Create and output for Public IP:
```hcl
output "WordPress_Public_IP" {
  description = "The public IP address of the EC2 instance"
  value       = module.wordpress_ec2.public_ip
}
```
Output name will be shown in console after applying terraform.

## Step 4: Innitialize and Apply Terraform
Initialize the Terraform project:
```bash
terraform init
```
Apply the Terraform configuration:
```bash
terraform apply
```
Confirm by typing yes when prompted. This will create an EC2 instance on AWS.

# Configuration Management with Ansible
Ansible is a powerful open-source automation tool that provides a simple and agentless way to automate IT tasks. 

## Step 1: Create Structure
This structure follows best practices in Ansible for organizing roles, variables, and configuration files. The separation of roles makes it modular and easy to manage. The staging directory suggests that there are different environments, and the Ansible configuration is tailored accordingly.
```bash
📦ansible
 ┣ 📂groups_vars
 ┃ ┗ 📂all
 ┃ ┃ ┗ 📜vars.yml
 ┣ 📂roles
 ┃ ┣ 📂mysql
 ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┣ 📜database.yml
 ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┗ 📜mysql_secure_installation.yml
 ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┗ 📜dotmy.cnf.j2
 ┃ ┗ 📂wordpress
 ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┣ 📜configure.yml
 ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┣ 📜httpd.conf.j2
 ┃ ┃ ┃ ┗ 📜wp-config.php.j2
 ┣ 📂staging
 ┃ ┗ 📜hosts
 ┣ 📜ansible.cfg
 ┣ 📜requirements.yml
 ┗ 📜wordpress.yml
```
- __group_vars__
  - __all:__
    - __vars.yml:__ This file likely contains variables that are common to all groups. Variables defined here will be applied to all hosts.
- __roles:__
  - __mysql:__
    - __handlers:__
      - __main.yml:__ This file contains Ansible handlers for the MySQL role. Handlers are triggered by tasks and can be used for tasks like restarting services.
    - __tasks:__
      - __database.yml:__ Task file for handling database-related operations.
      - __install.yml:__ Task file for installing MySQL.
      - __main.yml:__ Main task file for the MySQL role.
      - __mysql_secure_installation.yml:__ Task file for securing the MySQL installation.
    - __templates:__
      - __dotmy.cnf.j2:__ Jinja2 template for generating the MySQL configuration file.
  - __wordpress:__
    - __handlers:__
      - __main.yml:__ Handlers for the WordPress role.
    - __tasks:__
      - __configure.yml:__ Task file for configuring WordPress.
      - __install.yml:__ Task file for installing WordPress.
      - __main.yml:__ Main task file for the WordPress role.
    - __templates:__
      - __httpd.conf.j2:__ Jinja2 template for the Apache HTTP server configuration file.
      - __wp-config.php.j2:__ Jinja2 template for the WordPress configuration file.
- __staging:__
  - __hosts:__ Host inventory file for the staging environment.
- __ansible.cfg:__ Ansible configuration file for project-level settings.
- __requirements.yml:__ YAML file specifying external roles or collections required for the project.
- __wordpress.yml:__ The main playbook file for orchestrating the deployment. This YAML file likely includes plays and roles to be executed.

## Step 2: Configure ansible
Firstly, you need to create `ansible.cfg` file, which is an Ansible configuration file. We need to specify that it is a default profile, interpreter and paths to inventory file and roles directory (it is recomeded to mention full path).
```ini
[defaults]
interpreter_python = auto_silent
inventory = ./staging/hosts
roles_path = ./roles
```
Than we need to create `hosts` file in `staging` folder. It is an inventory file where you need to specify all hosts on which you will run playbooks. Here you need to specify host group, host name and IP of instances of which you need to execute playbooks.
```ini
[wordpress]
wp_server1 ansible_host=1.2.3.4
```

## Step 3: Create MySQL Role
This role will handle the installation and configuration of MySQL on your target servers.
### Handlers (roles/mysql/handlers/main.yml)
Handler is a special kind of task that gets executed when notified by other tasks. Handlers are typically used to perform actions like restarting a service or taking some other corrective action after a change has been made. Handlers are defined in the handlers directory within an Ansible role. 

In this case we need only two handlers:

- __Restart MySQL:__ This handler restarts the MySQL service.
- __Start and enable MySQL:__ This handler starts and enables the MySQL service.
```yaml
---
- name: Restart MySQL
  systemd:
    name: mysqld
    state: restarted
    daemon_reload: yes

- name: Start and enable MySQL
  systemd:
    name: mysqld
    state: started
    enabled: yes
```

### Main Task (roles/mysql/tasks/main.yml)
This task imports subtasks for installation (`install.yml`), secure installation (`mysql_secure_installation.yml`), and database setup (`database.yml`)
```yaml
---
- import_tasks: install.yml
- import_tasks: mysql_secure_installation.yml
- import_tasks: database.yml
```

### Install Task (roles/mysql/tasks/install.yml)
In this task we update the system and install all dependencies for MySQL.

- __Update the system:__ Uses dnf to update all packages, excluding the kernel.
- __Install MySQL and dependencies:__ Installs MySQL server, client, and other necessary packages.
- __Start and enable MySQL:__ Notifies the handler to start and enable the MySQL service
```yaml
---
- name: Update the system
  dnf:
    name: '*'
    state: latest
    exclude: kernel*

- name: Install MySQL and dependencies
  dnf:
    name:
      - mysql
      - mysql-server
      - python3-PyMySQL
      - python3-firewall
    state: latest
  notify: Start and enable MySQL
```

### Secure Install Task (roles/mysql/tasks/mysql_secure_installation.yml)
In this task we configure MySQL, set the root password and remove all unnecessary data.
- __Update root password for MySQL:__ Sets the root password for MySQL and configures access privileges.
- __Copy the root credentials as .my.cnf file:__ Copies the root credentials to a .my.cnf file for secure access.
- __Remove anonymous users:__ Removes anonymous MySQL users.
- __Remove test database:__ Removes the default test database
```yaml
---
- name: Update root password for MySQL
  community.mysql.mysql_user:
    login_user: root
    login_password: "root"
    name: root
    password: "{{ mysql_root_password }}"
    host: "{{ item }}"
    check_implicit_admin: yes
    priv: "*.*:ALL,GRANT"
  with_items:
    - "{{ ansible_hostname }}"
    - localhost
    - 127.0.0.1
    - ::1

- name: Copy the root credentials as .my.cnf file
  template:
    src: dotmy.cnf.j2
    dest: /root/.my.cnf
    mode: 0600

- name: Remove anonymous users
  community.mysql.mysql_user:
    name: ""
    host: "{{ item }}"
    state: absent
  with_items:
    - localhost
    - "{{ ansible_hostname }}"

- name: Remove test database
  community.mysql.mysql_db:
    name: test
    state: absent
```

### Database Task (roles/mysql/tasks/database.yml)
In this task we create the database and user for Wordpress.
- __Create database for WordPress:__ Creates a MySQL database for WordPress.
- __Create user for WordPress:__ Creates a MySQL user with the necessary privileges for WordPress.
- __Restart MySQL:__ Notifies the handler to restart the MySQL service
```yaml
---
- name: Create database for Wordpress
  community.mysql.mysql_db:
    name: "{{ wordpress_db_name }}"
    state: present
    login_user: root
    login_password: "{{ mysql_root_password }}"

- name: Create user for Wordpress
  community.mysql.mysql_user:
    name: "{{ wordpress_db_user }}"
    password: "{{ wordpress_db_password }}"
    priv: "{{ wordpress_db_name }}.*:ALL"
    host: "{{ wordpress_db_host }}"
    state: present
  notify: Restart MySQL
```

### Dotmy.cnf Template (roles/mysql/templates/dotmy.cnf.j2)
This is a Jinja2 template that generates a MySQL configuration file (.my.cnf) with the root user credentials from Ansible variables.
```yaml
[client]
user = root
password = {{ mysql_root_password }}
```

Ensure that you've set the appropriate variables like `mysql_root_password`, `wordpress_db_name`, `wordpress_db_user`, `wordpress_db_password`, and `wordpress_db_host` in your playbook or inventory file before running the role. We will do it in the end.

## Step 4: Create Wordpress Role
This step involves creating an Ansible role to set up and configure WordPress on your server. The role includes tasks to install dependencies, extract the WordPress archive, configure Apache, and more.

### Handlers (roles/wordpress/handlers/main.yml)
- __Restart Apache:__ This handler restarts the httpd service.
- __Start and enable Apache:__ This handler starts and enables the httpd service.

```yaml
---
- name: Restart Apache
  service:
    name: 'httpd'
    state: restarted
  become: true
  
- name: Start and Enable Apache
  service:
    name: 'httpd'
    state: started
    enabled: yes
  become: true
```

### Main Task (roles/wordpress/tasks/main.yml)
This main task includes two subtasks, `install.yml` and `configure.yml`.

```yaml
---
- include_tasks: install.yml
- include_tasks: configure.yml
```

### Install Task (roles/wordpress/tasks/install.yml)
This task installs system updates and necessary dependencies.
- __Update the system:__ Uses dnf to update all packages, excluding the kernel.
- __Install PHP and dependencies:__ Installs PHP, PHP Plugins and other necessary packages.
- __Start and enable Apache:__ Notifies the handler to start and enable the httpd service

```yaml
---
- name: Update the system
  dnf:
    name: '*'
    state: latest
    exclude: kernel*

- name: Install Dependencies
  dnf:
    name: "{{ item }}"
    state: latest
  loop: [ 'httpd', 'php', 'php-curl', 'php-gd', 'php-mbstring', 'php-xml', 'php-soap', 'php-intl', 'php-zip', 'php-mysqlnd' ]
  notify:
    - Start and Enable Apache
```

### Configure Task (roles/wordpress/tasks/configure.yml)
This task performs various configurations for WordPress, including extracting the archive, setting up Apache VirtualHost, and configuring the wp-config.php file.
- __Extract Wordpress:__ Downloads and extracts Wordpress archive.
- __Set up Apache VirtualHost:__ Configure Apache Web Server for Wordpress works correctly.
- __Copy wp-config.php file content:__ Configure Wordpress, by changing configuration file.
- __Ensure SELinux allows web server to write to WordPress:__ Allow httpd connection to MySQL, so Wordpress can use database.

```yaml
---
- name: Extract Wordpress
  unarchive:
    src: https://wordpress.org/latest.tar.gz
    dest: /var/www/html
    remote_src: yes

- name: Set up Apache VirtualHost
  template:
    src: "httpd.conf.j2"
    dest: "/etc/httpd/conf/httpd.conf "
  notify: Restart Apache

- name: Copy wp-config.php file content
  template:
    src: wp-config.php.j2
    dest: "/var/www/html/wordpress/wp-config.php"
  notify:
    - Restart Apache

- name: Ensure SELinux allows web server to write to WordPress
  seboolean:
    name: httpd_can_network_connect_db
    state: yes
```

### Httpd template (roles/wordpress/templates/httpd.cfg.j2)
This is a Jinja2 template for Apache VirtualHost configuration.

```apache
<VirtualHost *:80>
    ServerAdmin ec2-user@localhost
    DocumentRoot /var/www/html/wordpress
    # ... (other configurations)
</VirtualHost>
```

### Wordpress template (roles/wordpress/templates/wp-config.php.j2)
This is a Jinja2 template for the wp-config.php file.

```php
<?php
// ... (other configurations)
define('DB_NAME', '{{ wordpress_db_name }}');
define('DB_USER', '{{ wordpress_db_user }}');
define('DB_PASSWORD', '{{ wordpress_db_password }}');
define('DB_HOST', '{{ wordpress_db_host }}');
// ... (other configurations)
define('WP_HOME', 'http://{{ wordpress_server_ip }}/wordpress');
define('WP_SITEURL', 'http://{{ wordpress_server_ip }}/wordpress');
?>
```

These templates are used to dynamically generate configuration files during the Ansible playbook execution.

Ensure that you've set the appropriate variables like `wordpress_server_ip`, `wordpress_db_name`, `wordpress_db_user`, `wordpress_db_password`, and `wordpress_db_host` in your playbook or inventory file before running the role. We will do it in the end.

## Step 6: Provide variables
You need to specify variables in `groups_vars/all/vars.yml`. Use your own data, but if you want to run MySQL on Wordpress Server use localhost like in example. These variables is necessary for MySQL and Wordpress to start and comunicate with each other.

```yaml
---
mysql_root_password: root
mysql_bind_address: localhost
wordpress_server_ip: 1.2.3.4
wordpress_db_user: wordpress
wordpress_db_password: wordpress
wordpress_db_name: wp_db
wordpress_db_host: localhost
```
In near future I will provide a documentation about how to store variables in AWS Secrets Manager.

## Step 7: Run Playbook
Now that we have configured both the MySQL and WordPress roles, it's time to run the Ansible playbook to deploy and set up our WordPress environment on the target server. 

Execute the Ansible playbook using the following command:
```bash
ansible-playbook wordpress.yml --key-file="../.ssh/id_rsa"
```
This command tells Ansible to run the playbook (`wordpress.yml`) and use the (`--key-file="../.ssh/id_rsa"`) to provide path to SSH private key, so Ansible can connect to target server.

# Conclusion
To check if Wordpress is working follow this steps:
1. Open your web browser and navigate to the IP address or domain name of your target server.
```
http://your_server_ip/wordpress
```
Replace `your_server_ip` with the actual IP address or domain of your server.

2. Follow the WordPress setup wizard to complete the installation by providing the required information, including the site title, username, password, and email address.
3. After completing the setup, you should have a fully functional WordPress site accessible at the provided URL.

Congratulations! You have successfully deployed and configured WordPress using Terraform and Ansible.

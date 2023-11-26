# Launch a VM with Nagios that will monitor status of the docker from Task 4 and the VM from Task 3
# Introduction
This guide aims to walk you through the process of launching a virtual machine (VM) and installing Nagios and NRPE plugin on Amazon Web Services (AWS) instance using Terraform for infrastructure provisioning and Ansible for configuration management. 
Nagios and NRPE (Nagios Remote Plugin Executor) form a powerful combination for monitoring and managing the health of your infrastructure. Nagios is an open-source monitoring system that provides a centralized view of your entire IT infrastructure, while NRPE allows you to execute Nagios plugins on remote hosts.

# Why Nagios and NRPE?

1. __Comprehensive Monitoring:__ Nagios allows you to monitor hosts, services, and network devices, providing a comprehensive view of your infrastructure's health.
2. __Remote Execution:__ NRPE enables the execution of Nagios plugins on remote hosts. This is essential for monitoring specific metrics or services on distributed systems.
3. __Customizable Plugins:__ Nagios plugins are scripts or executables that perform specific checks. NRPE allows you to write or use existing plugins to monitor diverse aspects of your systems.

# Prerequisites

Before you begin working with Nagios and NRPE, ensure that you have the following prerequisites:

1. __Terraform Installation:__
Follow the official [Terraform installation guide](https://developer.hashicorp.com/terraform/install) to install Terraform on your machine.
2. __Ansible Installation:__
Follow the official [Ansible installation guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) to install Ansible.
3. __AWS Account:__
Create an AWS account if you don't have one already. This guide assumes you have AWS credentials configured on your machine. I will discuss how to provide credentials later.
4. __Infrastructure:__ 
Ensure you have done task 3 and task 4, and all servers working correctly. Also we will need code from this tasks to run Terraform and Ansible scripts.

# Overview of the Process:
1. __Terraform Configuration (Infrastructure Provisioning):__
Use Terraform to define the AWS resources needed, such as EC2 instances, security groups, and key pairs.
2. __Terraform Execution:__
Initialize the Terraform project and apply the configuration to create the specified AWS resources.
3. __Ansible Playbook (Configuration Management):__
Create an Ansible playbook to configure the EC2 instance, install necessary software (Apache, MySQL, PHP, and WordPress), and set up WordPress.
4. __Run Ansible Playbook:__
Execute the Ansible playbook to apply the configuration to the launched EC2 instance.
5. __Access Nagios:__
Open a web browser and access Nagios Core, so you can monitor all resources.
6. ___Optional: Resource Cleanup:___
If desired, use Terraform to destroy the AWS resources created during the process.

# Let's Get Started:
Proceed to the next sections to follow the step-by-step guide on launching a VM and installing Nagios with NRPE using Terraform and Ansible.

## Step 1: Create Terraform Configuration
### Create files and directories
Create a directory structure for this project, you can use next script to create the structure.

```bash
# Create directories
mkdir -p ansible/roles/nagios/handlers ansible/roles/nagios/tasks ansible/roles/nagios/templates 
mkdir -p ansible/roles/nrpe/handlers ansible/roles/nrpe/tasks ansible/roles/nrpe/templates

# Create Terraform files
touch ansible/roles/nagios/handlers/main.yml ansible/roles/nrpe/handlers/main.yml
touch ansible/roles/nagios/tasks/main.yml ansible/roles/nagios/tasks/install.yml ansible/roles/nagios/tasks/admin.yml ansible/roles/nagios/tasks/compile.yml ansible/roles/nagios/tasks/configure.yml ansible/roles/nagios/tasks/plugins.yml ansible/roles/nagios/tasks/nrpe.yml 
touch ansible/roles/nrpe/tasks/main.yml ansible/roles/nrpe/tasks/install.yml ansible/roles/nrpe/tasks/agent.yml ansible/roles/nrpe/tasks/plugins.yml ansible/roles/nrpe/tasks/configure.yml ansible/roles/nrpe/tasks/service.yml 
touch ansible/roles/nagios/templates/hosts.cfg.j2 ansible/roles/nagios/templates/services.cfg.j2 ansible/roles/nagios/templates/nagios.conf.j2
touch ansible/roles/nrpe/templates/nrpe.cfg.j2 ansible/roles/nrpe/templates/check_apache.j2 ansible/roles/nrpe/templates/check_disk_space.j2 ansible/roles/nrpe/templates/check_docker.j2 ansible/roles/nrpe/templates/check_processes.j2 ansible/roles/nrpe/templates/check_wordpress.j2
```

After execution you'll get next structure
```
📦 Project Directory
 ┣ 📂ansible
 ┃ ┗ 📂roles
 ┃ ┃ ┣ 📂nagios
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┣ 📜admin.yml
 ┃ ┃ ┃ ┃ ┣ 📜compile.yml
 ┃ ┃ ┃ ┃ ┣ 📜configure.yml
 ┃ ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┃ ┣ 📜nrpe.yml
 ┃ ┃ ┃ ┃ ┗ 📜plugins.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┣ 📜hosts.cfg.j2
 ┃ ┃ ┃ ┃ ┣ 📜nagios.conf.j2
 ┃ ┃ ┃ ┃ ┗ 📜services.cfg.j2
 ┃ ┃ ┣ 📂nrpe
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┣ 📜agent.yml
 ┃ ┃ ┃ ┃ ┣ 📜configure.yml
 ┃ ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┃ ┣ 📜plugins.yml
 ┃ ┃ ┃ ┃ ┗ 📜service.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┣ 📜check_apache.j2
 ┃ ┃ ┃ ┃ ┣ 📜check_disk_space.j2
 ┃ ┃ ┃ ┃ ┣ 📜check_docker.j2
 ┃ ┃ ┃ ┃ ┣ 📜check_processes.j2
 ┃ ┃ ┃ ┃ ┣ 📜check_wordpress.j2
 ┃ ┃ ┃ ┃ ┗ 📜nrpe.cfg.j2
 ┃ ┗ ...
 ┗ ...
```
This directory structure is organized for a Terraform project. Let's break down the contents:

- __📂nagios:__ Nagios role directory.
  - __📂handlers:__ Directory for Nagios role handlers.
    - __📜main.yml:__ Nagios role handlers definitions.
  - __📂tasks:__ Directory for Nagios role tasks.
    - __📜admin.yml:__ Tasks to create admin user for Nagios.
    - __📜compile.yml:__ Tasks related to compilation on Nagios Core.
    - __📜configure.yml:__ Tasks for configuration of Apache.
    - __📜install.yml:__ Tasks to update the system and install dependencies.
    - __📜main.yml:__ Main tasks that includes other tasks.
    - __📜nrpe.yml:__ Tasks related to NRPE setup.
    - __📜plugins.yml:__ Tasks for installing plugins.
  - __📂templates:__ Directory for Jinja2 templates used in Nagios configuration.
    - __📜hosts.cfg.j2:__ Template for hosts configuration.
    - __📜nagios.conf.j2:__ Template for Apache configuration.
    - __📜services.cfg.j2:__ Template for services configuration.
- __📂nrpe:__ NRPE role directory.
  - __📂handlers:__ Directory for NRPE role handlers.
    - __📜main.yml:__ NRPE role handler definitions.
  - __📂tasks:__ Directory for NRPE role tasks.
    - __📜agent.yml:__ Tasks for NRPE agent setup.
    - __📜configure.yml:__ Tasks for configuring NRPE.
    - __📜install.yml:__ Tasks to update the system and install dependencies.
    - __📜main.yml:__ Main tasks that includes other tasks.
    - __📜plugins.yml:__ Tasks for installing NRPE plugins.
    - __📜service.yml:__ Tasks for configuring NRPE service.
  - __📂templates:__ Directory for Jinja2 templates used in NRPE configuration.
    - __📜check_apache.j2:__ Template for Apache check.
    - __📜check_disk_space.j2:__ Template for disk space check.
    - __📜check_docker.j2:__ Template for Docker check.
    - __📜check_processes.j2:__ Template for processes check.
    - __📜check_wordpress.j2:__ Template for WordPress check.
    - __📜nrpe.cfg.j2:__ Template for NRPE configuration.

This structure suggests that the Ansible project is organized into roles, with each role having handlers, tasks, and templates directories for managing specific configurations and actions related to Nagios and NRPE. The templates directory contains Jinja2 templates used to generate configuration files.

## Step 2: Create module in root
In this step, we are creating a module in the root directory of your Terraform project. This module is named `nagios_ec2`, and it is defined using the configuration provided in the `main.tf` file. Let's break down the key components of this Terraform configuration:

```hcl
# main.tf
module "nagios_ec2" {
  source = "./modules/ec2"

  instance_name = "nagios-instance-1"
  ami           = var.image
  instance_type = "t2.micro"

  sg_name   = "nagios-sg"
  icmp      = true
  key_name  = "nagios"
  add_ports = [5666]
  key_path  = "${path.module}/.ssh/id_rsa.pub"
}
```
Explanation:

- __module "nagios_ec2":__ This declares a module named `nagios_ec2`. Modules are reusable units of Terraform configurations.
- __source = "./modules/ec2":__ Specifies the source directory of the module. In this case, the module is defined in the ./modules/ec2 directory.

__Module input variables:__
- __instance_name:__ Specifies the name of the EC2 instance as `nagios-instance-1`.
- __ami:__ Uses a variable `var.image` as the Amazon Machine Image (AMI) for the EC2 instance.
- __instance_type:__ Specifies the instance type as `t2.micro`.
- __sg_name:__ Specifies the name of the security group as `nagios-sg`.
- __icmp:__ Enables ICMP traffic in the security group.
key_name: Specifies the key pair name as `nagios`.
- __add_ports:__ Opens additional ports, such as port `5666`, in the security group.
- __key_path:__ Specifies the path to the public key file for SSH access.

This module is designed to create an EC2 instance for Nagios with the specified configuration parameters. It encapsulates the details of the EC2 instance, making the main Terraform configuration cleaner and more modular.

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
output "Nagios_Public_IP" {
  description = "The public IP address of the EC2 instance"
  value       = module.nagios_ec2.public_ip
}
```
Output name will be shown in console after applying terraform.

### Initialize and apply Terraform configuration
Initialize the Terraform project:
```bash
terraform init
```
Apply the Terraform configuration:
```bash
terraform apply
```
Confirm by typing yes when prompted. This will create an EC2 instance on AWS.

## Step 3: Create Nagios Role
This role configures Nagios Core on web server. Also it install NRPE plugin for communications from Nagios to agents.

```hcl
📂nagios
┣ 📂handlers
┃ ┗ 📜main.yml
┣ 📂tasks
┃ ┣ 📜admin.yml
┃ ┣ 📜compile.yml
┃ ┣ 📜configure.yml
┃ ┣ 📜install.yml
┃ ┣ 📜main.yml
┃ ┣ 📜nrpe.yml
┃ ┗ 📜plugins.yml
┗ 📂templates
┃ ┣ 📜hosts.cfg.j2
┃ ┣ 📜nagios.conf.j2
┃ ┗ 📜services.cfg.j2
```
Let's run throught the files, so we can understand what this role is doing.

### Handlers
We have two handlers for this role. Restart Apache and Restart Nagios. They are necessary for project, because without restart the configuration will not update.
```yaml
# nagios/handlers/main.yml
---
- name: Restart Apache
  service:
    name: httpd
    state: restarted

- name: Restart Nagios
  service:
    name: nagios
    state: restarted
```

### Main Task
```yaml
# nagios/tasks/main.yml
---
- name: Install Nagios Core and Plugins
  include_tasks: install.yml

- name: Configure Apache Web Server
  include_tasks: configure.yml

- name: Build and Install Nagios Core
  include_tasks: compile.yml

- name: Create Nagios Admin User
  include_tasks: admin.yml

- name: Build and Install Nagios Plugins
  include_tasks: plugins.yml

- name: Configure Nagios Core to use NRPE
  include_tasks: nrpe.yml
```

This main task file orchestrates the different aspects of setting up Nagios. It includes sub-tasks for installing Nagios Core and Plugins, configuring Apache, building and installing Nagios Core, creating the Nagios admin user, building and installing Nagios Plugins, and configuring Nagios Core to use NRPE.

### Install Task
```yaml
# nagios/tasks/install.yml
---
- name: Update the system
  dnf:
    name: '*'
    state: latest
    exclude: kernel*

- name: Install required packages
  dnf:
    name: "{{ item }}"
    state: present
  loop: [ "httpd", "docker", "php", "php-cli", "gcc", "glibc", "glibc-common", "gd", "gd-devel", "make", "net-snmp", "openssl", "perl", "wget", "unzip", "openssl-devel", "python-pip", "gettext", "automake", "autoconf", "net-snmp-utils" ]

- name: Install passlib
  pip:
    name: passlib
    executable: /usr/bin/pip3

- name: Download and Extract Nagios Core source
  ansible.builtin.unarchive:
    src: "https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-{{ nagios_version }}/nagios-{{ nagios_version }}.tar.gz"
    dest: "/tmp/"
    remote_src: yes

- name: Download and Extract Nagios Plugins
  ansible.builtin.unarchive:
    src: "https://github.com/nagios-plugins/nagios-plugins/archive/release-{{ nagios_plugins_version }}.tar.gz"
    dest: "/tmp"
    remote_src: yes
    validate_certs: no
    creates: "/tmp/nagios-plugins-release-{{ nagios_plugins_version }}"
```

This task file performs the installation of Nagios and its dependencies. It includes tasks for updating the system, installing required packages, installing the passlib Python library, and downloading and extracting Nagios Core and Plugins.

### Apache Configuration Task
```yaml
# nagios/tasks/configure.yml
---
- name: Configure Apache for Nagios
  template:
    src: "nagios.conf.j2"
    dest: "/etc/httpd/conf.d/nagios.conf"
  notify: Restart Apache
```

This task file configures Apache to work with Nagios. It uses a Jinja2 template (`nagios.conf.j2`) to generate the Apache configuration file for Nagios and notifies Apache to restart after configuration changes.

### Build Nagios Task
```yaml
# nagios/tasks/compile.yml
---
- name: Configure Nagios Core
  command: "./configure --with-httpd-conf=/etc/httpd/conf.d"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Install Nagios Core
  command: "make all"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Install Nagios Core users files
  command: "make install-groups-users"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Add user apache to group nagios
  user:
    name: apache
    groups: nagios
    append: yes

- name: Install Nagios Core files
  command: "make install"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Install Nagios daemon init
  command: make install-daemoninit
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Install Nagios Command Mode
  command: "make install-commandmode"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"

- name: Install Nagios Configuration Files
  command: "make install-config"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"  

- name: Install Apache Config Files
  command: "make install-webconf"
  args:
    chdir: "/tmp/nagios-{{ nagios_version }}"
  notify: 
    - Restart Apache
    - Restart Nagios
```

This task file handles the compilation and installation of Nagios Core. It includes tasks for configuring Nagios, installing Nagios Core and its user files, adding the Apache user to the Nagios group, and installing various Nagios components.

### Admin User Task
```yaml
# nagios/tasks/admin.yml
---
- name: Create htpasswd file
  htpasswd:
    path: /usr/local/nagios/etc/htpasswd.users
    name: "{{ nagios_admin_user }}"
    password: "{{ nagios_admin_password }}"
    state: present
  notify: 
    - Restart Apache
    - Restart Nagios
```

This task file creates an htpasswd file for Nagios admin user authentication. It uses the `htpasswd` Ansible module to manage the user and password, and it notifies Apache and Nagios to restart after the htpasswd file is created.

### Plugins Task
```yaml
# nagios/tasks/plugins.yml
---
- name: Sutup Nagios Plugins
  command: "./tools/setup"
  args:
    chdir: "/tmp/nagios-plugins-release-{{ nagios_plugins_version }}"

- name: Compile Nagios Plugins
  command: "./configure"
  args:
    chdir: "/tmp/nagios-plugins-release-{{ nagios_plugins_version }}"

- name: Init Nagios Plugins
  command: "make"
  args:
    chdir: "/tmp/nagios-plugins-release-{{ nagios_plugins_version }}"

- name: Install Nagios Plugins
  command: "make install"
  args:
    chdir: "/tmp/nagios-plugins-release-{{ nagios_plugins_version }}"
  notify:
    - Restart Apache
    - Restart Nagios
```

This task file is responsible for setting up and installing Nagios Plugins. It includes tasks for running the setup script, compiling the plugins, initializing them, and finally, installing them.

 It also notifies Apache and Nagios to restart after the plugins are installed.

### NRPE Task
```yaml
# nagios/tasks/nrpe.yml
---
- name: Add check_nrpe command to Nagios
  blockinfile:
    path: /usr/local/nagios/etc/objects/commands.cfg
    block: |
      define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
      }
    insertafter: EOF

- name: Add hosts to Nagios
  template:
    src: templates/hosts.cfg.j2
    dest: /usr/local/nagios/etc/hosts.cfg
    owner: "{{ nagios_user }}"
    group: "{{ nagios_group }}"
    mode: 0644

- name: Add services to Nagios
  template:
    src: templates/services.cfg.j2
    dest: /usr/local/nagios/etc/services.cfg
    owner: "{{ nagios_user }}"
    group: "{{ nagios_group }}"
    mode: 0644

- name: Add hosts and services to Nagios config
  lineinfile:
    path: /usr/local/nagios/etc/nagios.cfg
    line: "{{ item }}"
    insertafter: EOF
    state: present
  loop:
    - "cfg_file=/usr/local/nagios/etc/hosts.cfg"
    - "cfg_file=/usr/local/nagios/etc/services.cfg"
  notify: Restart Nagios
```

This task file handles the configuration of NRPE for Nagios. It adds the `check_nrpe` command to Nagios, sets up hosts and services configuration files using Jinja2 templates, and includes them in the Nagios configuration. It also notifies Nagios to restart after the configuration changes.

### Templates

#### `hosts.cfg.j2`
```jinja
define host{
    name                            linux
    use                             generic-host           
    check_period                    24x7        
    check_interval                  5       
    retry_interval                  1       
    max_check_attempts              10      
    check_command                   check-host-alive
    notification_period             24x7    
    notification_interval           30      
    notification_options            d,r     
    contact_groups                  admins  
    register                        0                     
}

define host{
    use                             linux
    host_name                       wordpress
    alias                           RHEL 9 Instance with Wordpress
    address                         {{ wordpress_server_ip }}
}

define host{
    use                             linux
    host_name                       docker
    alias                           RHEL 9 Instance with Docker and Apache
    address                         {{ docker_server_ip }}
}
```
- This template defines Nagios host configurations for two hosts: `wordpress` and `docker`. It includes common settings for Linux hosts and specific details for each host, such as alias and IP address.

#### `nagios.conf.j2`
```jinja
Alias /nagios "/usr/local/nagios/share"

<Directory "/usr/local/nagios/share">
    Options None
    AllowOverride None
    Require all granted
</Directory>

ScriptAlias /nagios/cgi-bin "/usr/local/nagios/sbin"

<Directory "/usr/local/nagios/sbin">
    Options ExecCGI
    AllowOverride None
    Require all granted
</Directory>
```
- This template configures Apache to serve Nagios web interface. It sets up aliases and script aliases for Nagios directories and specifies directory options.

#### `services.j2`
```jinja
define service{
    use                     generic-service
    host_name               wordpress
    service_description     CPU Load
    check_command           check_nrpe!check_load
}

define service{
    use                     generic-service
    host_name               wordpress
    service_description     Total Processes
    check_command           check_nrpe!check_total_procs
}

# ... (other services for the WordPress host)

define service{
    use                     generic-service
    host_name               docker
    service_description     Check if Docker is running
    check_command           check_nrpe!check_docker 
}

# ... (other services for the Docker host)
```
- This template defines Nagios service configurations for various checks on the `wordpress` and `docker` hosts. It includes services for CPU load, total processes, web server status, MySQL process, WordPress website, disk space, HTTP, Docker status, etc.

These templates are essential for generating Nagios configurations dynamically based on the specific hosts and services in your infrastructure. They provide a flexible and maintainable way to configure and scale your Nagios monitoring setup.

## Step 4: Create NRPE role

### Handlers
```yaml
# nrpe/handlers/main.yml
---
- name: Enable NRPE Service
  service:
    name: nrpe
    enabled: yes
    state: started
  notify: Restart NRPE Service

- name: Restart NRPE Service
  service:
    name: nrpe
    state: restarted
```

1. __Enable NRPE Service:__
This handler enables the NRPE service on the target host. The notify keyword triggers the `Restart NRPE Service` handler whenever this handler runs.

2. __Restart NRPE Service:__
This handler restarts the NRPE service on the target host. It's triggered by the `Enable NRPE Service` handler, ensuring that the service is restarted after any configuration changes.

The `notify` keyword ensures that the `Restart NRPE Service` handler is only executed when the `Enable NRPE Service` handler makes changes. This prevents unnecessary restarts and improves overall performance.

### Main task
```yaml
# nrpe/tasks/main.yml
---
- name: Install Nagios NRPE Agent and Plugins
  include_tasks: install.yml

- name: Build Nagios NRPE Agent
  include_tasks: agent.yml

- name: Build Nagios NRPE Plugins
  include_tasks: plugins.yml

- name: Configure Nagios NRPE Agent and Plugins
  include_tasks: configure.yml

- name: Configure Nagios NRPE service
  include_tasks: service.yml
```
This main task file orchestrates the different aspects of setting up NRPE Agent on server. It includes sub-tasks for installing NRPE Agent and Plugins, configuring Apache, building and installing NRPE Agent, and configuring NRPE Service.

### Install task
```yaml
# nrpe/tasks/install.yml
---
- name: Install Required Build Packages/Tools
  yum:
    name:
      - gcc
      - glibc
      - glibc-common
      - openssl
      - openssl-devel
    state: present

- name: Download and Extract NRPE Source File 
  unarchive:
    src: "https://github.com/NagiosEnterprises/nrpe/archive/nrpe-{{ nrpe_version }}.tar.gz"
    dest: /tmp
    remote_src: yes
    validate_certs: no

- name: Download and Extract NRPE Plugins Source File 
  unarchive:
    src: "https://github.com/nagios-plugins/nagios-plugins/releases/download/release-{{ nrpe_plugin_version }}/nagios-plugins-{{ nrpe_plugin_version }}.tar.gz"
    dest: /tmp
    remote_src: yes
    validate_certs: no
```
- This task installs the essential build packages and tools for compiling the NRPE agent and plugins. These packages provide the necessary development environment for building software from source. 
- Downloads the NRPE source code from the specified GitHub repository. It utilizes the `unarchive` module to extract the source code to the `/tmp` directory. 
- Downloads the NRPE plugins source code from the specified GitHub repository. It similarly uses the `unarchive` module to extract the plugin source code to the `/tmp` directory. 

These tasks ensure that the necessary build tools and source files are available for building the NRPE agent and plugins. They provide the foundation for the subsequent tasks that handle the actual compilation and configuration.

### NRPE Agent Configuration task
```yaml
# nrpe/tasks/agent.yml
---
- name: Configure Nagios NRPE Agent
  command: ./configure --enable-command-args 
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Build Nagios NRPE Agent
  command: make all
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install Nagios NRPE Agent Groups and Users
  command: make install-groups-users
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install Nagios NRPE Agent
  command: make install
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install Nagios NRPE Agent Config
  command: make install-config
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Initialize Nagios Agent
  command: make install-init
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}
  notify: Restart NRPE Service
```
- This task runs the `./configure` command with the `--enable-command-args` option, which enables support for command-line arguments. This allows the NRPE agent to accept additional parameters when executed. 
- Executes the `make all` command, which builds the NRPE agent executable. This compiles the source code into a binary file that can be run by the system. 
- Runs the `make install-groups-users` command, which creates the necessary groups and users for the NRPE agent to run properly. This ensures that the agent has the required permissions to access resources.
- Executes the `make install` command, which installs the compiled NRPE agent executable and other related files to their designated locations. This places the agent in the correct directory for execution.
- Runs the `make install-config` command, which copies the NRPE agent configuration files to their proper locations. This ensures that the agent is configured with the desired settings.
- Executes the `make install-init` command, which initializes the NRPE agent and sets up the necessary system services. This prepares the agent for operation.
- Notifies the `"Restart NRPE Service"` handler when any of the preceding tasks make changes. The handler will then restart the NRPE service to apply the new configuration.

These tasks perform the complete process of configuring, building, installing, and initializing the Nagios NRPE agent. They ensure that the agent is properly set up and ready to monitor the system.

### NRPE Plaugins task
```yaml
# nrpe/tasks/plugins.yml
---
- name: Configure Nagios NRPE Plugins
  command: ./configure --enable-command-args
  args:
    chdir: /tmp/nagios-plugins-{{ nrpe_plugin_version }}

- name: Build Nagios NRPE Plugins
  command: make all
  args:
    chdir: /tmp/nagios-plugins-{{ nrpe_plugin_version }}

- name: Install Nagios NRPE Plugins
  command: make install
  args:
    chdir: /tmp/nagios-plugins-{{ nrpe_plugin_version }}
```
- This task runs the `./configure` command with the `--enable-command-args` option, which enables support for command-line arguments. This allows the NRPE plugins to accept additional parameters when executed.
- Executes the `make all` command, which builds the NRPE plugins binaries. This compiles the source code into executable files that can be used by the NRPE agent.
- Runs the `make install` command, which installs the compiled NRPE plugins binaries and other related files to their designated locations. This places the plugins where the NRPE agent can find them.

These tasks perform the complete process of configuring, building, and installing the Nagios NRPE plugins. They ensure that the plugins are properly set up and ready to be used by the NRPE agent to monitor the system.

### Configure NRPE task
```yaml
# nrpe/tasks/configure.yml
---
- name: Add NRPE Config
  template:
    src: nrpe.cfg.j2
    dest: /usr/local/nagios/etc/nrpe.cfg
    user: "{{ nrpe_user }}"
    group: "{{ nrpe_group }}"
  notify: Restart NRPE Service
```
This task utilizes the template module to generate and place the NRPE configuration file `(nrpe.cfg)`. It uses the `nrpe.cfg.j2` Jinja2 template as the source and places the generated configuration file in `/usr/local/nagios/etc/nrpe.cfg`. The user and group options specify the ownership of the generated file.

It is dynamically generates and places the NRPE configuration file, ensuring that the agent is configured with the desired settings. The Jinja2 template allows for flexibility in defining the configuration based on variables and templates. The notification to the handler ensures that the service is restarted to reflect the new configuration.

### NRPE Service task
```yaml
# nrpe/tasks/main.yml
---
- name: Update services file with NRPE Service Name, Port and Protocol
  lineinfile:
    path: /etc/services
    line: "nrpe            5666/tcp                # NRPE Service"

- name: Disable SSL when running NRPE Agent 
  replace:
    path: /usr/lib/systemd/system/nrpe.service
    regexp: '-f$'
    replace: '-f -n'
```
- This task utilizes the `lineinfile` module to add the NRPE service definition to the `/etc/services` file. It ensures that the NRPE service is listed and properly defined.
- Uses the `replace` module to modify the NRPE service configuration file `/usr/lib/systemd/system/nrpe.service`. It replaces the `-f` flag with `-f -n`, disabling SSL when running the NRPE agent. This prevents potential SSL-related issues during agent operation.

### Templates

#### `check_apache.j2`
```jinja
#!/bin/bash

if systemctl is-active httpd >/dev/null 2>&1; then
  echo "OK: Web server is running"
  exit 0
else
  echo "CRITICAL: Web server is not running"
  exit 2
fi
```
This script checks the status of the Apache web server on a Linux system using the `systemctl` command. It exits with a zero code if the HTTP service is active and running, indicating a healthy state. If the HTTP service is not active, it exits with a non-zero code, signifying a critical error and a non-functional web server.

#### `check_disk_space.j2`
```jinja
#!/bin/bash

threshold=80
current_usage=$(df -h /var/www/html/wordpress/  | awk 'NR==2 {print $5}' | cut -d'%' -f1)

if [ "$current_usage" -lt "$threshold" ]; then
  echo "OK: Disk space usage is below $threshold%"
  exit 0
else
  echo "CRITICAL: Disk space usage is above $threshold%"
  exit 2
fi
```
This script monitors the disk space usage of the `/var/www/html/wordpress/` directory, setting a threshold of 80%. It checks the current usage using the `df -h` command and compares it to the threshold. If the current usage is below the threshold, it exits with a zero code, indicating a healthy state. If the current usage exceeds the threshold, it exits with a non-zero code, signifying a critical error and insufficient disk space for the WordPress installation.

#### `check_docker.j2`
```jinja
#!/bin/bash

OK=0
WARNING=1
CRITICAL=2
UNKNOWN=3

CONTAINER_NAME=""
DOCKER_CMD="docker"
HTTPD_SERVICE_NAME=""

usage() {
    echo "Usage: $0 -c <container_name> [-s <httpd_service_name>] [-d <docker_command>]"
    exit $UNKNOWN
}

while getopts "c:d:s:" opt; do
    case $opt in
        c)
            CONTAINER_NAME="$OPTARG"
            ;;
        d)
            DOCKER_CMD="$OPTARG"
            ;;
        s)
            HTTPD_SERVICE_NAME="$OPTARG"
            ;;
        \?)
            usage
            ;;
    esac
done

if [ -z "$CONTAINER_NAME" ]; then
    echo "DOCKER UNKNOWN - Container name is required."
    usage
fi

if [ -n "$HTTPD_SERVICE_NAME" ]; then
    RUNNING=$($DOCKER_CMD inspect --format '{% raw %}{{.State.Running}}{% endraw %}' "$CONTAINER_NAME" 2>/dev/null)

    if [ "$RUNNING" == "true" ]; then
        # Check if HTTPD service is running inside the container
        HTTPD_RUNNING=$($DOCKER_CMD exec "$CONTAINER_NAME" pgrep "$HTTPD_SERVICE_NAME" 2>/dev/null)

        if [ -n "$HTTPD_RUNNING" ]; then
            echo "DOCKER OK - Container $CONTAINER_NAME is running and $HTTPD_SERVICE_NAME is running inside."
            exit $OK
        else
            echo "DOCKER CRITICAL - Container $CONTAINER_NAME is running, but $HTTPD_SERVICE_NAME is not running inside."
            exit $CRITICAL
        fi
    elif [ "$RUNNING" == "false" ]; then
        echo "DOCKER CRITICAL - Container $CONTAINER_NAME is not running."
        exit $CRITICAL
    else
        echo "DOCKER UNKNOWN - Unable to determine the state of container $CONTAINER_NAME."
        exit $UNKNOWN
    fi
else
    RUNNING=$($DOCKER_CMD inspect --format '{% raw %}{{.State.Running}}{% endraw %}' "$CONTAINER_NAME" 2>/dev/null)

    if [ "$RUNNING" == "true" ]; then
        echo "DOCKER OK - Container $CONTAINER_NAME is running."
        exit $OK
    elif [ "$RUNNING" == "false" ]; then
        echo "DOCKER CRITICAL - Container $CONTAINER_NAME is not running."
        exit $CRITICAL
    else
        echo "DOCKER UNKNOWN - Unable to determine the state of container $CONTAINER_NAME."
        exit $UNKNOWN
    fi
fi
```
This script monitors the status of a Docker container and an optional HTTP service running within it. It utilizes the `docker` command to inspect the container's state and check if the specified HTTP service is running.

**Script Breakdown:**

1. **Define Exit Codes:**

   The script defines exit codes for different statuses: OK (0), WARNING (1), CRITICAL (2), and UNKNOWN (3). These codes are used to communicate the container's health to Nagios.

2. **Parse Command-Line Arguments:**

   The `getopts` function is used to parse command-line arguments, extracting the container name (`-c`), Docker command (`-d`), and HTTP service name (`-s`).

3. **Validate Container Name:**

   If the container name is not provided, the script displays an error message and exits with the UNKNOWN code.

4. **Check Container Status:**

   Using the `docker inspect` command, the script checks if the specified container is running. If it's not running, the script exits with the CRITICAL code.

5. **Check HTTP Service Status (Optional):**

   If the HTTP service name is provided, the script checks if the specified service is running inside the container using the `docker exec` command. If the service is not running, the script exits with the CRITICAL code.

6. **Return Appropriate Exit Code:**

   Based on the container's state and HTTP service status (if applicable), the script returns the corresponding exit code: OK for a healthy container with the service running, CRITICAL for a non-running container or service, and UNKNOWN for any unexpected conditions.

#### `check_processes.j2`
```jinja
#!/bin/bash

if pgrep -x "mysqld" >/dev/null 2>&1; then
  echo "OK: MySQL process is running"
  exit 0
else
  echo "CRITICAL: MySQL process is not running"
  exit 2
fi
```
This script monitors the presence of the MySQL process using the `pgrep` command. It checks if the mysqld process is running, indicating that the MySQL server is operational.

#### `check_wordpress.j2`
```jinja
#!/bin/bash

if curl -sSf http://localhost/wordpress >/dev/null; then
  echo "OK: WordPress website is accessible"
  exit 0
else
  echo "CRITICAL: WordPress website is not accessible"
  exit 2
fi
```
This script checks the accessibility of a WordPress installation running on the local machine. It uses the `curl` command to send a silent request to the WordPress site's homepage `(http://localhost/wordpress)`.


#### `nrpe.cfg.j2`
```jinja
allowed_hosts=127.0.0.1,{{ nagios_server_ip }}

...

dont_blame_nrpe=1

...

command[check_apache]=/bin/bash /usr/local/nagios/libexec/check_apache
command[check_processes]=/bin/bash /usr/local/nagios/libexec/check_processes
command[check_wordpress]=/bin/bash /usr/local/nagios/libexec/check_wordpress
command[check_disk_space]=/bin/bash /usr/local/nagios/libexec/check_disk_space
command[check_http]=/bin/bash /usr/local/nagios/libexec/check_http -I localhost

command[check_docker]=/bin/bash /usr/local/nagios/libexec/check_docker -c apache-container
command[check_docker_httpd]=/bin/bash /usr/local/nagios/libexec/check_docker -c apache-container -s httpd
```
This template defines the configuration parameters for the Nagios NRPE agent. It specifies the allowed hosts, disables NRPE blaming, and defines commands for various monitoring checks.

**Key Configuration Points:**

- `allowed_hosts`: Restricts access to the NRPE agent to specific IP addresses. In this case, it allows connections from `127.0.0.1` (localhost) and the `nagios_server_ip` variable.

- `dont_blame_nrpe`: Disables NRPE blaming, preventing NRPE from taking responsibility for errors that occur in the plugins it executes.

- `command[check_apache]`, `command[check_processes]`, etc.: Define the paths to the executable scripts for various monitoring checks, including checking the Apache web server, running processes, the WordPress website, disk space usage, and HTTP accessibility.

- `command[check_docker]`, `command[check_docker_httpd]`: Define the paths to the executable scripts for monitoring Docker containers, specifically the `apache-container` and checking the HTTP service running within that container.

## Step 5: Create playbooks
In this step you need to create playbook for Nagios server, but you don't need any playbook for NRPE Plugin, in case you can oly specify NRPE role in Wordpress playbook. Here are scripts to do this.

### `nagios.yml`
```yaml
---
- name: Install and Compile Nagios Core 
  hosts: nagios
  user: ec2-user
  become: yes
  vars_files:
    - nagios

  roles:
    - nagios
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `nagios` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable files to load for the playbook. In this case, the `nagios` variable file is loaded.
- `roles`: Defines the roles to include in the playbook. In this case, the `nagios` and `nrpe` roles are included.

**Role Breakdown:**

- `nagios`: Presumably contains tasks for installing and configuring the Nagios Core monitoring server.
- `nrpe`: Presumably contains tasks for installing and configuring the Nagios Remote Plugin Executor (NRPE) agent.

**Overall Purpose:**

The playbook serves as a high-level orchestration of the installation and configuration of the Nagios Core monitoring server and the NRPE agent. It utilizes roles to encapsulate the specific tasks for each component and relies on variable files to provide environment-specific configuration parameters.

### `wordpress.yml`
```yaml
---
- name: Install and Launch Wordpress 
  hosts: wordpress
  user: ec2-user
  become: yes
  vars_files:
    - mysql
    - wordpress
    - nagios

  roles:
    - mysql
    - wordpress
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `wordpress` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable files to load for the playbook. In this case, the `mysql`, `wordpress`, and `nagios` variable files are loaded.
- `roles`: Defines the roles to include in the playbook. In this case, the `mysql`, `wordpress`, and `nrpe` roles are included.

**Role Breakdown:**

- `mysql`: Presumably contains tasks for installing and configuring the MySQL database server, which is the backend database for WordPress.
- `wordpress`: Presumably contains tasks for installing, configuring, and launching the WordPress application, including creating the WordPress database and setting up the WordPress website.
- `nrpe`: Presumably contains tasks for installing and configuring the Nagios Remote Plugin Executor (NRPE) agent, which enables monitoring of the WordPress server.

**Overall Purpose:**

The playbook serves as a high-level orchestration of the installation and configuration of the MySQL database server, the WordPress application, and the NRPE agent on the target host. It utilizes roles to encapsulate the specific tasks for each component and relies on variable files to provide environment-specific configuration parameters.

This playbook effectively sets up a complete WordPress environment, including the database server, the WordPress application itself, and the monitoring capabilities provided by NRPE.

### `docker.yml`
```yaml
---
- name: Install NRPE on Docker Hosts 
  hosts: docker
  user: ec2-user
  become: yes
  vars_files:
    - nagios

  roles:
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `docker` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable file to load for the playbook. In this case, the `nagios` variable file is loaded.
- `roles`: Defines the role to include in the playbook. In this case, the `nrpe` role is included.

**Role Breakdown:**

- `nrpe`: Presumably contains tasks for installing and configuring the Nagios Remote Plugin Executor (NRPE) agent on the Docker hosts. This allows for monitoring of the Docker containers and their associated applications.

**Overall Purpose:**

The playbook serves as a high-level orchestration of the installation and configuration of the NRPE agent on the target host, which is assumed to be managing Docker containers. This enables monitoring of the Docker environment and its components using Nagios.

The playbook effectively extends the monitoring capabilities to Docker hosts, allowing for comprehensive oversight of the containerized applications running on them.

### `site.yml`
```yaml
---
- name: Configure Wordpres server with NRPE
  import_playbook: wordpress.yml

- name: Configure Docker server with NRPE
  import_playbook: docker.yml

- name: Configure Nagios server with NRPE
  import_playbook: nagios.yml
```
**Playbook Structure:**

- `name`: Defines the name of the playbook, which is "Configure Wordpres server with NRPE" in this case.
- `import_playbook`: Imports other Ansible playbooks to execute their tasks within this playbook.

**Imported Playbooks:**

1. `wordpress.yml`: This playbook is imported to install and configure the MySQL database server, the WordPress application, and the NRPE agent on the target host, presumably setting up a complete WordPress environment.

2. `docker.yml`: This playbook is imported to install and configure the NRPE agent on the Docker hosts, enabling monitoring of the Docker containers and their associated applications.

3. `nagios.yml`: This playbook is imported to install and configure the Nagios Core monitoring server and the NRPE agent on the target host, establishing the central monitoring system.

**Overall Purpose:**

The `site.yml` playbook serves as a central orchestrator for configuring and deploying the entire monitoring infrastructure, encompassing the WordPress server, Docker server, and Nagios server. It effectively utilizes the imported playbooks to streamline the setup of each component and their integration with the Nagios monitoring system.

By importing these playbooks, the `site.yml` playbook simplifies the overall configuration process and ensures that each component is properly set up and integrated with the Nagios monitoring framework. This approach promotes consistency and ease of management for the monitoring infrastructure.

## Step 6: Run Ansible Playbook
Before we start, I will show you how invetory file is looks.
```ini
[wordpress]
wp_server1 ansible_host=1.2.3.4

[nagios]
ng_server1 ansible_host=5.6.7.8

[docker]
dk_server1 ansible_host=9.1.2.3

[all:vars]
ansible_ssh_private_key_file=./.ssh/id_rsa
```
This Ansible inventory file defines the hosts to be managed by Ansible playbooks and their associated hostnames and IP addresses. It also specifies the default SSH private key file to use for connecting to the hosts.

**Host Groups:**

- `wordpress`: This group contains the WordPress server with the hostname `wp_server1` and IP address.

- `nagios`: This group contains the Nagios server with the hostname `ng_server1` and IP address.

- `docker`: This group contains the Docker server with the hostname `dk_server1` and IP address.

**Default SSH Private Key File:**

- `ansible_ssh_private_key_file=./.ssh/id_rsa`: This variable defines the default SSH private key file to use for connecting to the hosts. It is set to `./.ssh/id_rsa`, indicating that the file is located in the current directory and named id_rsa. This file contains the private key that Ansible will use to authenticate with the hosts when executing tasks.

This inventory file provides Ansible with the necessary information to connect to and manage the WordPress, Nagios, and Docker servers. It defines the host groups, hostnames, IP addresses, and the default SSH private key file, enabling Ansible to perform its tasks effectively.

### Run Playbook
You need to run `site.yml` playbook.To do this, follow these steps:

1. **Navigate to the playbook directory:** Use the `cd` command to navigate to the directory containing the `site.yml` playbook.

2. **Execute the playbook:** Run the following command to execute the `site.yml` playbook:

```bash
ansible-playbook site.yml
```

This command will execute the playbook, importing the sub-playbooks `wordpress.yml`, `docker.yml`, and `nagios.yml`, and applying their configurations to the respective hosts defined in the inventory file.

4. **Verify the configurations:** Once the playbook execution is complete, verify that the WordPress, Docker, and Nagios servers have been configured as expected. You can check the logs or use Ansible's ad-hoc commands to confirm the configurations.

5. **Access Nagios:** In the address bar of the web browser, type the URL of the Nagios server. The exact URL will depend on your specific setup, but it typically follows the format `http://<nagios_server_hostname_or_IP_address>/nagios`. For example. Once you've entered the URL and pressed Enter, you'll be prompted for login credentials. The default username is `nagiosadmin`, and the default password is `nagiosadmin`.

## Conclusion
Creating Ansible roles for Nagios and NRPE provides a modular and reusable approach to deploying and managing Nagios monitoring infrastructure. By encapsulating the configuration tasks and variables for each component into separate roles, you can achieve greater flexibility, maintainability, and scalability in your monitoring setup.

The Nagios role can be used to install and configure the Nagios Core monitoring server, including defining host templates, services, and command checks. The NRPE role can be used to install and configure the Nagios Remote Plugin Executor (NRPE) agent on remote hosts, enabling them to communicate with the Nagios server and provide monitoring data.

Utilizing these roles streamlines the deployment process, ensures consistent configurations, and simplifies future maintenance. The modularity of roles allows you to easily adapt and extend the monitoring setup to suit your specific needs and environment.

By incorporating these roles into your Ansible playbooks, you can effectively establish a comprehensive and reliable monitoring infrastructure for your network devices, servers, and applications.
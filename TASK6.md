# Launch a VM with Prometheus that will monitor status of the docker from task #4 and the VM from task #3
# Introduction
Prometheus is an open-source monitoring and alerting toolkit designed for reliability and scalability. It is widely used for monitoring containerized applications and microservices architectures. Prometheus collects metrics from various targets such as containers, applications, and VMs, processes and stores them in a time-series database. With its powerful querying language (PromQL), alerting capabilities, and integration with Grafana, Prometheus provides a comprehensive monitoring solution.

# Why Preometheus?
1. **Data Model:** Prometheus uses a multi-dimensional data model, allowing metrics to be stored with key-value pairs. This flexibility makes it easy to filter, aggregate, and query metrics.
2. **Pull-Based Architecture:** Prometheus follows a pull-based model, where it scrapes metrics from targets at regular intervals. This approach is suitable for dynamic environments like those with containers.
3. **Service Discovery:** Prometheus supports service discovery mechanisms, making it adaptable to dynamic environments. It can automatically discover and monitor new instances as they come online.
4. **Alerting:** Prometheus has a built-in alerting system that can trigger alerts based on predefined rules. It supports integration with various notification channels.
5. **Ecosystem Integration:** Prometheus integrates seamlessly with Grafana, allowing users to create rich, interactive dashboards for visualization. This combination provides a powerful and flexible monitoring solution.

Now, let's proceed with the steps to launch a VM with Prometheus that will monitor the status of Docker containers and the VM itself.

# Prerequisites

Before you begin working with prometheus and NRPE, ensure that you have the following prerequisites:

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
5. __Access Prometheus:__
Open a web browser and access Prometheus, so you can monitor all resources.
6. ___Optional: Resource Cleanup:___
If desired, use Terraform to destroy the AWS resources created during the process.

# Let's Get Started:
Proceed to the next sections to follow the step-by-step guide on launching a VM and installing Prometheus using Terraform and Ansible.

## Step 1: Create files and directories
Create a directory structure for this project, you can use next script to create the structure.

```bash
mkdir -p ansible/roles/apache_exporter/{handlers,tasks,templates}
mkdir -p ansible/roles/docker_exporter/{handlers,tasks,templates}
mkdir -p ansible/roles/node_exporter/{handlers,tasks,templates}
mkdir -p ansible/roles/prometheus/{handlers,tasks,templates}

# Create files inside the roles
touch ansible/roles/apache_exporter/handlers/main.yml
touch ansible/roles/apache_exporter/tasks/{binary.yml,httpd.yml,install.yml,main.yml,service.yml,user.yml}
touch ansible/roles/apache_exporter/templates/apache_exporter.service.j2

touch ansible/roles/docker_exporter/handlers/main.yml
touch ansible/roles/docker_exporter/tasks/main.yml
touch ansible/roles/docker_exporter/templates/daemon.json.j2

touch ansible/roles/node_exporter/handlers/main.yml
touch ansible/roles/node_exporter/tasks/{binary.yml,install.yml,main.yml,service.yml,user.yml}
touch ansible/roles/node_exporter/templates/node_exporter.service.j2

touch ansible/roles/prometheus/handlers/main.yml
touch ansible/roles/prometheus/tasks/{configure.yml,install.yml,main.yml,permissions.yml,service.yml}
touch ansible/roles/prometheus/templates/{prometheus.service.j2,prometheus.yml.j2}
```

After execution you'll get next structure
```
📦 Project Directory
 ┣ 📂ansible
 ┃ ┗ 📂roles
 ┃ ┃ ┣ 📂apache_exporter
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┣ 📜binary.yml
 ┃ ┃ ┃ ┃ ┣ 📜httpd.yml
 ┃ ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┃ ┣ 📜service.yml
 ┃ ┃ ┃ ┃ ┗ 📜user.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┗ 📜apache_exporter.service.j2
 ┃ ┃ ┣ 📂docker_exporter
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┗ 📜daemon.json.j2
 ┃ ┃ ┣ 📂node_exporter
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┣ 📜binary.yml
 ┃ ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┃ ┣ 📜service.yml
 ┃ ┃ ┃ ┃ ┗ 📜user.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┗ 📜node_exporter.service.j2
 ┃ ┃ ┣ 📂prometheus
 ┃ ┃ ┃ ┣ 📂handlers
 ┃ ┃ ┃ ┃ ┗ 📜main.yml
 ┃ ┃ ┃ ┣ 📂tasks
 ┃ ┃ ┃ ┃ ┣ 📜configure.yml
 ┃ ┃ ┃ ┃ ┣ 📜install.yml
 ┃ ┃ ┃ ┃ ┣ 📜main.yml
 ┃ ┃ ┃ ┃ ┣ 📜permisions.yml
 ┃ ┃ ┃ ┃ ┗ 📜service.yml
 ┃ ┃ ┃ ┗ 📂templates
 ┃ ┃ ┃ ┃ ┣ 📜prometheus.service.j2
 ┃ ┃ ┃ ┃ ┗ 📜prometheus.yml.j2
 ┃ ┗ ...
 ┗ ...
```
This directory structure is organized for a Terraform project. Let's break down the contents:

- __apache_exporter:__ Ansible role for Apache Exporter.
  - __handlers:__ Directory for Ansible handlers, which are tasks triggered by plays but only run once, after all the tasks in a particular play have been completed.
    - __main.yml:__ The handler file for Apache Exporter.
  - __tasks:__ Directory for Ansible tasks.
    - __binary.yml:__ Task file for handling binary-related tasks for Apache Exporter.
    - __httpd.yml:__ Task file for handling tasks related to Apache HTTP server.
    - __install.yml:__ Task file for installing Apache Exporter.
    - __main.yml:__ Main task file for Apache Exporter.
    - __service.yml:__ Task file for configuring the Apache Exporter service.
    - __user.yml:__ Task file for handling user-related tasks for Apache Exporter.
  - __templates:__ Directory for Jinja2 templates used by the role.
    - __apache_exporter.service.j2:__ Jinja2 template for the Apache Exporter systemd service file.
- __docker_exporter:__ Ansible role for Docker Exporter.
  - __handlers:__ Directory for Ansible handlers.
    - __main.yml:__ The handler file for Docker Exporter.
  - __tasks:__ Directory for Ansible tasks.
    - __main.yml:__ Main task file for Docker Exporter.
  - __templates:__ Directory for Jinja2 templates.
    - __daemon.json.j2:__ Jinja2 template for the Docker daemon configuration file.
- __node_exporter:__ Ansible role for Node Exporter.
  - __handlers:__ Directory for Ansible handlers.
    - __main.yml:__ The handler file for Node Exporter.
  - __tasks:__ Directory for Ansible tasks.
    - __binary.yml:__ Task file for handling binary-related tasks for Node Exporter.
    - __install.yml:__ Task file for installing Node Exporter.
    - __main.yml:__ Main task file for Node Exporter.
    - __service.yml:__ Task file for configuring the Node Exporter service.
    - __user.yml:__ Task file for handling user-related tasks for Node Exporter.
  - __templates:__ Directory for Jinja2 templates.
    - __node_exporter.service.j2:__ Jinja2 template for the Node Exporter systemd service file.
- __prometheus:__ Ansible role for Prometheus.
  - __handlers:__ Directory for Ansible handlers.
    - __main.yml:__ The handler file for Prometheus.
  - __tasks:__ Directory for Ansible tasks.
    - __configure.yml:__ Task file for configuring Prometheus.
    - __install.yml:__ Task file for installing Prometheus.
    - __main.yml:__ Main task file for Prometheus.
    - __permissions.yml:__ Task file for setting permissions for Prometheus.
    - __service.yml:__ Task file for configuring the Prometheus service.
  - __templates:__ Directory for Jinja2 templates.
    - __prometheus.service.j2:__ Jinja2 template for the Prometheus systemd service file.
    - __prometheus.yml.j2:__ Jinja2 template for the Prometheus configuration file.

This directory structure follows the conventions of an Ansible project with roles organized by functionality and common directories like handlers, tasks, and templates for better organization and maintainability.

## Step 2: Create module in root
In this step, we are creating a module in the root directory of your Terraform project. This module is named `prometheus_ec2`, and it is defined using the configuration provided in the `main.tf` file. Let's break down the key components of this Terraform configuration:

```hcl
# main.tf
module "prometheus_ec2" {
  source = "./modules/ec2"

  instance_name = "prometheus-instance-1"
  ami           = var.image
  instance_type = "t2.micro"

  sg_name   = "prometheus-sg"
  icmp      = true
  key_name  = "prometheus"
  add_ports = [9090]
  key_path  = "${path.module}/.ssh/id_rsa.pub"
}
```
Explanation:

- __module "prometheus_ec2":__ This declares a module named `prometheus_ec2`. Modules are reusable units of Terraform configurations.
- __source = "./modules/ec2":__ Specifies the source directory of the module. In this case, the module is defined in the ./modules/ec2 directory.

__Module input variables:__
- __instance_name:__ Specifies the name of the EC2 instance as `prometheus-instance-1`.
- __ami:__ Uses a variable `var.image` as the Amazon Machine Image (AMI) for the EC2 instance.
- __instance_type:__ Specifies the instance type as `t2.micro`.
- __sg_name:__ Specifies the name of the security group as `prometheus-sg`.
- __icmp:__ Enables ICMP traffic in the security group.
- __key_name:__ Specifies the key pair name as `prometheus`.
- __add_ports:__ Opens additional ports, such as port `9090`, in the security group.
- __key_path:__ Specifies the path to the public key file for SSH access.

This module is designed to create an EC2 instance for prometheus with the specified configuration parameters. It encapsulates the details of the EC2 instance, making the main Terraform configuration cleaner and more modular.

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
output "prometheus_Public_IP" {
  description = "The public IP address of the EC2 instance"
  value       = module.prometheus_ec2.public_ip
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

## Step 3: Create Prometheus Role
This role configures Prometheus on web server.

```hcl
 📂prometheus
 ┣ 📂handlers
 ┃ ┗ 📜main.yml
 ┣ 📂tasks
 ┃ ┣ 📜configure.yml
 ┃ ┣ 📜install.yml
 ┃ ┣ 📜main.yml
 ┃ ┣ 📜permisions.yml
 ┃ ┗ 📜service.yml
 ┗ 📂templates
 ┃ ┣ 📜prometheus.service.j2
 ┃ ┗ 📜prometheus.yml.j2
```
Let's run throught the files, so we can understand what this role is doing.

### Handlers
```yaml
# prometheus/handlers/main.yml
---
- name: Enable Prometheus service
  systemd:
    name: prometheus
    enabled: yes
    daemon_reload: yes
    state: started

- name: Restart Prometheus service
  systemd:
    name: prometheus
    state: restarted
```
1. **Enable Prometheus service:** This task enables the systemd service named prometheus, ensuring that the Prometheus daemon starts automatically when the system boots up. It also triggers a daemon reload to apply the configuration changes.

2. **Restart Prometheus service:** This task restarts the Prometheus service, which gracefully shuts down the existing Prometheus process and launches a new one with the updated configuration. This is useful for applying configuration changes without downtime.

The provided handlers defines two tasks related to the management of the Prometheus systemd service: enabling the service to start automatically and restarting the service to apply configuration changes. These tasks ensure that Prometheus is running and collecting metrics effectively.

### Main Task
```yaml
# prometheus/tasks/main.yml
---
- name: Install Prometheus and Dependencies
  include_tasks: install.yml

- name: Configure permissions
  include_tasks: permisions.yml

- name: Configure Prometheus
  include_tasks: configure.yml

- name: Configure Prometheus service
  include_tasks: service.yml
```
This task serves as a high-level orchestration of the different aspects of setting up Prometheus. Each included task file (`install.yml`, `permissions.yml`, `configure.yml`, and `service.yml`) focuses on a specific aspect of the setup process, promoting modularity and maintainability in the Ansible role.

### Install Task
```yaml
# prometheus/tasks/install.yml
---
- name: Update the system
  dnf:
    name: '*'
    state: latest
    exclude: kernel*
    
- name: Install Prometheus
  unarchive:
    src: https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz
    dest: /tmp
    remote_src: yes
```
- This task uses the dnf Ansible module to update the system packages (`state: latest`). It specifies that all packages should be updated (`name: '*'`) except for those matching the exclusion pattern `kernel*`.
- Uses the `unarchive` Ansible module to download and extract the Prometheus binary from the specified URL. The `src` parameter specifies the source URL of the Prometheus binary release, and the dest parameter specifies the destination directory (`/tmp`) where the archive should be extracted. The `remote_src: yes` option indicates that the source file is remote.

### Permisions Task
```yaml
# prometheus/tasks/permisions.yml
---
- name: Add prometheus group
  group:
    name: "{{ prometheus_group }}"
    state: present

- name: Add prometheus user
  user:
    name: "{{ prometheus_user }}"
    shell: /bin/false
    createhome: no

- name: Create Prometheus directories in "/etc" and "/var/lib" locations
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}" 
    mode: '0755'
  loop:
    - /etc/prometheus
    - /var/lib/prometheus

- name: Copy prometheus binary to /usr/local/bin
  copy:
    src: /tmp/prometheus-{{ prometheus_version }}.linux-amd64/prometheus
    dest: /usr/local/bin/prometheus
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}"
    mode: '0755'
    remote_src: yes

- name: Copy promtool binary to /usr/local/bin
  copy:
    src: /tmp/prometheus-{{ prometheus_version }}.linux-amd64/promtool
    dest: /usr/local/bin/promtool
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}"
    mode: '0755'
    remote_src: yes

- name: Copy consoles directoriy to /etc/prometheus
  copy:
    src: /tmp/prometheus-{{ prometheus_version }}.linux-amd64/consoles
    dest: /etc/prometheus/consoles
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}"
    mode: '0755'
    remote_src: yes

- name: Copy console_libraries directory to /etc/prometheus
  copy:
    src: /tmp/prometheus-{{ prometheus_version }}.linux-amd64/console_libraries
    dest: /etc/prometheus/console_libraries
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}"
    mode: '0755'
    remote_src: yes
```
The `permisions.yml` file contains tasks related to setting up permissions and creating necessary directories for Prometheus.

- This task uses the `group` Ansible module to ensure that the specified Prometheus group (`{{ prometheus_group }}`) exists on the system.

- Uses the `user` Ansible module to ensure that the specified Prometheus user (`{{ prometheus_user }}`) exists on the system. It sets the user's shell to `/bin/false` and specifies not to create a home directory.

- Uses the `file` Ansible module to create two directories: `/etc/prometheus` and `/var/lib/prometheus`. It ensures that both directories have the specified ownership (`{{ prometheus_user }}:{{ prometheus_group }}`) and permission mode (`0755`).

- Uses the `copy` Ansible module to copy the Prometheus binary from the temporary directory to `/usr/local/bin`. It sets the ownership, group, and permission mode for the copied file.

- Copies the `promtool` binary to `/usr/local/bin` with the specified ownership, group, and permission mode.

- Copies the `consoles` directory to `/etc/prometheus` with the specified ownership, group, and permission mode.

- Copies the `console_libraries` directory to `/etc/prometheus` with the specified ownership, group, and permission mode.

In summary, the task is responsible for setting up the necessary permissions, creating directories, and copying Prometheus-related binaries and configurations to their designated locations.

### Configure Prometheus Task
```yaml
# prometheus/tasks/configure.yml
---
- name: Create prometheus.yml file from template
  template:
    src: prometheus.yml.j2
    dest: /etc/prometheus/prometheus.yml
    owner: "{{ prometheus_user }}"
    group: "{{ prometheus_group }}"
    mode: '0644'
```
This task uses the template Ansible module to create the prometheus.yml configuration file from a Jinja2 template (`prometheus.yml.j2`). The template is processed, and the resulting file is placed in the destination path `/etc/prometheus/prometheus.yml`. The ownership, group, and file permissions are set accordingly.

### Service Task
```yaml
# prometheus/tasks/service.yml
---
- name: Copy Prometheus service configuration
  template:
    src: prometheus.service.j2
    dest: /etc/systemd/system/prometheus.service
  notify: 
    - Enable Prometheus service
    - Restart Prometheus service
```
This task is responsible for copying the Prometheus service configuration file (`prometheus.service.j2`) to the destination path `/etc/systemd/system/prometheus`.service. Additionally, it includes notifications to trigger other handlers or tasks when this task is executed.


### Templates

#### `prometheus.yml.j2`
```jinja
```
#### `prometheus.service.j2`
```jinja
```


## Step 4: Apache

### Handlers
```yaml
# nrpe/handlers/main.yml
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
- name: Install prometheus NRPE Agent and Plugins
  include_tasks: install.yml

- name: Build prometheus NRPE Agent
  include_tasks: agent.yml

- name: Build prometheus NRPE Plugins
  include_tasks: plugins.yml

- name: Configure prometheus NRPE Agent and Plugins
  include_tasks: configure.yml

- name: Configure prometheus NRPE service
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
    src: "https://github.com/prometheusEnterprises/nrpe/archive/nrpe-{{ nrpe_version }}.tar.gz"
    dest: /tmp
    remote_src: yes
    validate_certs: no

- name: Download and Extract NRPE Plugins Source File 
  unarchive:
    src: "https://github.com/prometheus-plugins/prometheus-plugins/releases/download/release-{{ nrpe_plugin_version }}/prometheus-plugins-{{ nrpe_plugin_version }}.tar.gz"
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
- name: Configure prometheus NRPE Agent
  command: ./configure --enable-command-args 
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Build prometheus NRPE Agent
  command: make all
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install prometheus NRPE Agent Groups and Users
  command: make install-groups-users
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install prometheus NRPE Agent
  command: make install
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Install prometheus NRPE Agent Config
  command: make install-config
  args:
    chdir: /tmp/nrpe-nrpe-{{ nrpe_nrpe_version }}

- name: Initialize prometheus Agent
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

These tasks perform the complete process of configuring, building, installing, and initializing the prometheus NRPE agent. They ensure that the agent is properly set up and ready to monitor the system.

### NRPE Plaugins task
```yaml
# nrpe/tasks/plugins.yml
---
- name: Configure prometheus NRPE Plugins
  command: ./configure --enable-command-args
  args:
    chdir: /tmp/prometheus-plugins-{{ nrpe_plugin_version }}

- name: Build prometheus NRPE Plugins
  command: make all
  args:
    chdir: /tmp/prometheus-plugins-{{ nrpe_plugin_version }}

- name: Install prometheus NRPE Plugins
  command: make install
  args:
    chdir: /tmp/prometheus-plugins-{{ nrpe_plugin_version }}
```
- This task runs the `./configure` command with the `--enable-command-args` option, which enables support for command-line arguments. This allows the NRPE plugins to accept additional parameters when executed.
- Executes the `make all` command, which builds the NRPE plugins binaries. This compiles the source code into executable files that can be used by the NRPE agent.
- Runs the `make install` command, which installs the compiled NRPE plugins binaries and other related files to their designated locations. This places the plugins where the NRPE agent can find them.

These tasks perform the complete process of configuring, building, and installing the prometheus NRPE plugins. They ensure that the plugins are properly set up and ready to be used by the NRPE agent to monitor the system.

### Configure NRPE task
```yaml
# nrpe/tasks/configure.yml
---
- name: Add NRPE Config
  template:
    src: nrpe.cfg.j2
    dest: /usr/local/prometheus/etc/nrpe.cfg
    user: "{{ nrpe_user }}"
    group: "{{ nrpe_group }}"
  notify: Restart NRPE Service
```
This task utilizes the template module to generate and place the NRPE configuration file `(nrpe.cfg)`. It uses the `nrpe.cfg.j2` Jinja2 template as the source and places the generated configuration file in `/usr/local/prometheus/etc/nrpe.cfg`. The user and group options specify the ownership of the generated file.

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

   The script defines exit codes for different statuses: OK (0), WARNING (1), CRITICAL (2), and UNKNOWN (3). These codes are used to communicate the container's health to prometheus.

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
allowed_hosts=127.0.0.1,{{ prometheus_server_ip }}

...

dont_blame_nrpe=1

...

command[check_apache]=/bin/bash /usr/local/prometheus/libexec/check_apache
command[check_processes]=/bin/bash /usr/local/prometheus/libexec/check_processes
command[check_wordpress]=/bin/bash /usr/local/prometheus/libexec/check_wordpress
command[check_disk_space]=/bin/bash /usr/local/prometheus/libexec/check_disk_space
command[check_http]=/bin/bash /usr/local/prometheus/libexec/check_http -I localhost

command[check_docker]=/bin/bash /usr/local/prometheus/libexec/check_docker -c apache-container
command[check_docker_httpd]=/bin/bash /usr/local/prometheus/libexec/check_docker -c apache-container -s httpd
```
This template defines the configuration parameters for the prometheus NRPE agent. It specifies the allowed hosts, disables NRPE blaming, and defines commands for various monitoring checks.

**Key Configuration Points:**

- `allowed_hosts`: Restricts access to the NRPE agent to specific IP addresses. In this case, it allows connections from `127.0.0.1` (localhost) and the `prometheus_server_ip` variable.

- `dont_blame_nrpe`: Disables NRPE blaming, preventing NRPE from taking responsibility for errors that occur in the plugins it executes.

- `command[check_apache]`, `command[check_processes]`, etc.: Define the paths to the executable scripts for various monitoring checks, including checking the Apache web server, running processes, the WordPress website, disk space usage, and HTTP accessibility.

- `command[check_docker]`, `command[check_docker_httpd]`: Define the paths to the executable scripts for monitoring Docker containers, specifically the `apache-container` and checking the HTTP service running within that container.

## Step 5: Create playbooks
In this step you need to create playbook for prometheus server, but you don't need any playbook for NRPE Plugin, in case you can oly specify NRPE role in Wordpress playbook. Here are scripts to do this.

### `prometheus.yml`
```yaml
---
- name: Install and Compile prometheus Core 
  hosts: prometheus
  user: ec2-user
  become: yes
  vars_files:
    - prometheus

  roles:
    - prometheus
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `prometheus` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable files to load for the playbook. In this case, the `prometheus` variable file is loaded.
- `roles`: Defines the roles to include in the playbook. In this case, the `prometheus` and `nrpe` roles are included.

**Role Breakdown:**

- `prometheus`: Presumably contains tasks for installing and configuring the prometheus Core monitoring server.
- `nrpe`: Presumably contains tasks for installing and configuring the prometheus Remote Plugin Executor (NRPE) agent.

**Overall Purpose:**

The playbook serves as a high-level orchestration of the installation and configuration of the prometheus Core monitoring server and the NRPE agent. It utilizes roles to encapsulate the specific tasks for each component and relies on variable files to provide environment-specific configuration parameters.

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
    - prometheus

  roles:
    - mysql
    - wordpress
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `wordpress` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable files to load for the playbook. In this case, the `mysql`, `wordpress`, and `prometheus` variable files are loaded.
- `roles`: Defines the roles to include in the playbook. In this case, the `mysql`, `wordpress`, and `nrpe` roles are included.

**Role Breakdown:**

- `mysql`: Presumably contains tasks for installing and configuring the MySQL database server, which is the backend database for WordPress.
- `wordpress`: Presumably contains tasks for installing, configuring, and launching the WordPress application, including creating the WordPress database and setting up the WordPress website.
- `nrpe`: Presumably contains tasks for installing and configuring the prometheus Remote Plugin Executor (NRPE) agent, which enables monitoring of the WordPress server.

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
    - prometheus

  roles:
    - nrpe
```
**Playbook Structure:**

- `hosts`: Defines the target host for the playbook, which is `docker` in this case.
- `user`: Specifies the user to run the playbook tasks as, which is `ec2-user` in this case.
- `become`: Indicates whether to escalate privileges using sudo or su. In this case, `become: yes` means that tasks will run with elevated privileges.
- `vars_files`: Specifies the variable file to load for the playbook. In this case, the `prometheus` variable file is loaded.
- `roles`: Defines the role to include in the playbook. In this case, the `nrpe` role is included.

**Role Breakdown:**

- `nrpe`: Presumably contains tasks for installing and configuring the prometheus Remote Plugin Executor (NRPE) agent on the Docker hosts. This allows for monitoring of the Docker containers and their associated applications.

**Overall Purpose:**

The playbook serves as a high-level orchestration of the installation and configuration of the NRPE agent on the target host, which is assumed to be managing Docker containers. This enables monitoring of the Docker environment and its components using prometheus.

The playbook effectively extends the monitoring capabilities to Docker hosts, allowing for comprehensive oversight of the containerized applications running on them.

### `site.yml`
```yaml
---
- name: Configure Wordpres server with NRPE
  import_playbook: wordpress.yml

- name: Configure Docker server with NRPE
  import_playbook: docker.yml

- name: Configure prometheus server with NRPE
  import_playbook: prometheus.yml
```
**Playbook Structure:**

- `name`: Defines the name of the playbook, which is "Configure Wordpres server with NRPE" in this case.
- `import_playbook`: Imports other Ansible playbooks to execute their tasks within this playbook.

**Imported Playbooks:**

1. `wordpress.yml`: This playbook is imported to install and configure the MySQL database server, the WordPress application, and the NRPE agent on the target host, presumably setting up a complete WordPress environment.

2. `docker.yml`: This playbook is imported to install and configure the NRPE agent on the Docker hosts, enabling monitoring of the Docker containers and their associated applications.

3. `prometheus.yml`: This playbook is imported to install and configure the prometheus Core monitoring server and the NRPE agent on the target host, establishing the central monitoring system.

**Overall Purpose:**

The `site.yml` playbook serves as a central orchestrator for configuring and deploying the entire monitoring infrastructure, encompassing the WordPress server, Docker server, and prometheus server. It effectively utilizes the imported playbooks to streamline the setup of each component and their integration with the prometheus monitoring system.

By importing these playbooks, the `site.yml` playbook simplifies the overall configuration process and ensures that each component is properly set up and integrated with the prometheus monitoring framework. This approach promotes consistency and ease of management for the monitoring infrastructure.

## Step 6: Run Ansible Playbook
Before we start, I will show you how invetory file is looks.
```ini
[wordpress]
wp_server1 ansible_host=1.2.3.4

[prometheus]
ng_server1 ansible_host=5.6.7.8

[docker]
dk_server1 ansible_host=9.1.2.3

[all:vars]
ansible_ssh_private_key_file=./.ssh/id_rsa
```
This Ansible inventory file defines the hosts to be managed by Ansible playbooks and their associated hostnames and IP addresses. It also specifies the default SSH private key file to use for connecting to the hosts.

**Host Groups:**

- `wordpress`: This group contains the WordPress server with the hostname `wp_server1` and IP address.

- `prometheus`: This group contains the prometheus server with the hostname `ng_server1` and IP address.

- `docker`: This group contains the Docker server with the hostname `dk_server1` and IP address.

**Default SSH Private Key File:**

- `ansible_ssh_private_key_file=./.ssh/id_rsa`: This variable defines the default SSH private key file to use for connecting to the hosts. It is set to `./.ssh/id_rsa`, indicating that the file is located in the current directory and named id_rsa. This file contains the private key that Ansible will use to authenticate with the hosts when executing tasks.

This inventory file provides Ansible with the necessary information to connect to and manage the WordPress, prometheus, and Docker servers. It defines the host groups, hostnames, IP addresses, and the default SSH private key file, enabling Ansible to perform its tasks effectively.

### Run Playbook
You need to run `site.yml` playbook.To do this, follow these steps:

1. **Navigate to the playbook directory:** Use the `cd` command to navigate to the directory containing the `site.yml` playbook.

2. **Execute the playbook:** Run the following command to execute the `site.yml` playbook:

```bash
ansible-playbook site.yml
```

This command will execute the playbook, importing the sub-playbooks `wordpress.yml`, `docker.yml`, and `prometheus.yml`, and applying their configurations to the respective hosts defined in the inventory file.

4. **Verify the configurations:** Once the playbook execution is complete, verify that the WordPress, Docker, and prometheus servers have been configured as expected. You can check the logs or use Ansible's ad-hoc commands to confirm the configurations.

5. **Access prometheus:** In the address bar of the web browser, type the URL of the prometheus server. The exact URL will depend on your specific setup, but it typically follows the format `http://<prometheus_server_hostname_or_IP_address>/prometheus`. For example. Once you've entered the URL and pressed Enter, you'll be prompted for login credentials. The default username is `prometheusadmin`, and the default password is `prometheusadmin`.

## Conclusion
Creating Ansible roles for prometheus and NRPE provides a modular and reusable approach to deploying and managing prometheus monitoring infrastructure. By encapsulating the configuration tasks and variables for each component into separate roles, you can achieve greater flexibility, maintainability, and scalability in your monitoring setup.

The prometheus role can be used to install and configure the prometheus Core monitoring server, including defining host templates, services, and command checks. The NRPE role can be used to install and configure the prometheus Remote Plugin Executor (NRPE) agent on remote hosts, enabling them to communicate with the prometheus server and provide monitoring data.

Utilizing these roles streamlines the deployment process, ensures consistent configurations, and simplifies future maintenance. The modularity of roles allows you to easily adapt and extend the monitoring setup to suit your specific needs and environment.

By incorporating these roles into your Ansible playbooks, you can effectively establish a comprehensive and reliable monitoring infrastructure for your network devices, servers, and applications.
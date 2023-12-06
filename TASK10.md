# Enhanced Ansible and Terraform Configuration

## Introduction

In this iteration of the infrastructure code, we have introduced improvements to both Ansible and Terraform configurations. The primary focus is on incorporating variables and utilizing Terraform data blocks for better code organization, readability, and maintainability. Additionally, AWS Secrets Manager is utilized to securely store sensitive information.

## Terraform Code Enhancements

### 1. Variables and Data Blocks

In `variables.tf`, we've introduced variables to make the code more flexible and customizable. In `data.tf`, we've utilized data blocks to fetch information from AWS Secrets Manager.

#### variables.tf

```hcl
variable "region" {
  default     = "us-east-1"
  type        = string
  description = "The AWS region to deploy to"
}

variable "secrets_arn" {
  type        = string
  description = "The ARN of the Secrets Manager secret"
}
```

#### data.tf

```hcl
data "aws_secretsmanager_secret" "secrets" {
  arn = var.secrets_arn
}

data "aws_secretsmanager_secret_version" "current" {
  secret_id = data.aws_secretsmanager_secret.secrets.id
}

locals {
  secret = jsondecode(data.aws_secretsmanager_secret_version.current.secret_string)
}

data "aws_ami" "wordpress_latest" {
  most_recent = true

  filter {
    name   = "name"
    values = [local.secret["wordpress_ami_name"]]
  }

  owners = [local.secret["ami_owner"]]
}

data "aws_ami" "docker_latest" {
  most_recent = true

  filter {
    name   = "name"
    values = [local.secret["docker_ami_name"]]
  }

  owners = [local.secret["ami_owner"]]
}

data "aws_ami" "prometheus_latest" {
  most_recent = true

  filter {
    name   = "name"
    values = [local.secret["prometheus_ami_name"]]
  }

  owners = [local.secret["ami_owner"]]
}
```

**Changes Made:**
- Introduced variables for AWS region and Secrets Manager ARN for better customization.
- Utilized data blocks to securely fetch sensitive information from AWS Secrets Manager.

## Ansible Configuration Enhancements

### 1. Group Variables

In `group_vars/all/all.yml`, we've incorporated variable references for better organization and readability.

#### group_vars/all/all.yml

```yaml
---
nagios_server_ip: "{{ hostvars['ng_server1']['ansible_host'] }}"
wordpress_server_ip: "{{ hostvars['wp_server1']['ansible_host'] }}"
docker_server_ip: "{{ hostvars['dk_server1']['ansible_host'] }}"

wordpress_table_prefix: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/wordpress.wordpress_table_prefix', nested=true) }}"
wordpress_debug: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/wordpress.wordpress_debug', nested=true) }}"

mysql_root_password: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.mysql_root_password', nested=true) }}"
mysql_bind_address: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.mysql_bind_address', nested=true) }}"
wordpress_db_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.wordpress_db_user', nested=true) }}"
wordpress_db_password: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.wordpress_db_password', nested=true) }}"
wordpress_db_name: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.wordpress_db_name', nested=true) }}"
wordpress_db_host: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/mysql.wordpress_db_host', nested=true) }}"

nagios_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_version', nested=true) }}"
nagios_plugins_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_plugins_version', nested=true) }}"
nagios_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_user', nested=true) }}"
nagios_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_group', nested=true) }}"
nagios_admin_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_admin_user', nested=true) }}"
nagios_admin_password: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nagios_admin_password', nested=true) }}"

nrpe_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nrpe_version', nested=true) }}"
plugin_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.plugin_version', nested=true) }}"
nrpe_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nrpe_user', nested=true) }}"
nrpe_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/nagios.nrpe_group', nested=true) }}"

prometheus_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.prometheus_version', nested=true) }}"
prometheus_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.prometheus_user', nested=true) }}"
prometheus_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.prometheus_group', nested=true) }}"

apache_exporter_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.apache_exporter_user', nested=true) }}"
apache_exporter_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.apache_exporter_group', nested=true) }}"
apache_exporter_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.apache_exporter_version', nested=true) }}"

node_exporter_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.node_exporter_version', nested=true) }}"
node_exporter_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.node_exporter_user', nested=true) }}"
node_exporter_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.node_exporter_group', nested=true) }}"

alert_manager_version: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.alert_manager_version', nested=true) }}"
alert_manager_user: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.alert_manager_user', nested=true) }}"
alert_manager_group: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.alert_manager_group', nested=true) }}"
alert_manager_checksum: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.alert_manager_checksum', nested=true) }}"
telegram_api_token: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.telegram_api_token', nested=true) }}"
telegram_chat_id: "{{ lookup('amazon.aws.aws_secret', 'dev/ansible/prometheus.telegram_chat_id', nested=true) }}"
```

**Changes Made:**
- Replaced hardcoded values with variable references for better maintainability.
- Sensitive information is fetched securely from AWS Secrets Manager.

## Conclusion

By introducing variables, utilizing data blocks, and fetching sensitive information securely from AWS Secrets Manager, the codebase becomes more modular, flexible, and secure. These enhancements contribute to better code organization, readability, and ease of maintenance
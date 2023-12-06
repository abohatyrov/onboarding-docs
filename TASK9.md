# Ansible AWS Dynamic Inventory Configuration

## Introduction

Ansible provides a dynamic inventory system that allows you to manage your infrastructure in a flexible and automated way. One of the popular dynamic inventory plugins for Ansible is the `aws_ec2` plugin, specifically designed for working with Amazon EC2 instances in AWS. This documentation guides you through the configuration of the `aws_ec2` dynamic inventory plugin for AWS instances in Ansible.

### AWS EC2 Plugin Overview

The `aws_ec2` plugin enables Ansible to dynamically discover and manage AWS EC2 instances based on defined criteria. It leverages the AWS SDK for Python (Boto3) to interact with AWS services. This plugin allows you to filter instances based on various attributes such as region, instance state, and tags.

## AWS Credentials Configuration

Before using the `aws_ec2` plugin, ensure that your AWS credentials are properly configured. Ansible relies on the Boto3 library, which looks for AWS credentials in standard locations like environment variables, AWS CLI configuration files, or IAM roles assigned to an EC2 instance.

### Configuration Options:

1. **Environment Variables:**
   Set the following environment variables with your AWS credentials:

   ```bash
   export AWS_ACCESS_KEY_ID=your_access_key
   export AWS_SECRET_ACCESS_KEY=your_secret_key
   ```

2. **AWS CLI Configuration:**
   Ensure you have the AWS CLI installed and configured with the necessary credentials:

   ```bash
   aws configure
   ```

   Follow the prompts to input your AWS Access Key ID, Secret Access Key, default region, and output format.

3. **IAM Role (for EC2 Instances):**
   If running Ansible on an EC2 instance, you can assign an IAM role to the instance with the required permissions for AWS operations.

## Configuration

### Inventory File (aws_ec2.yml)

The inventory file, `aws_ec2.yml`, is where you define the configuration for the `aws_ec2` dynamic inventory plugin. Below is an example configuration:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1

filters:
  instance-state-name: running
  tag: Ansible: true

keyed_groups:
  - key: tags.Application
    separator: ''

compose:
  ansible_host: public_dns_name
```

#### Explanation of Configuration:

- `plugin`: Specifies the dynamic inventory plugin to use (`amazon.aws.aws_ec2` in this case).
- `regions`: Lists the AWS regions to query for instances (e.g., `us-east-1`).
- `filters`: Defines filters to narrow down the instances, such as filtering by instance state and tags.
- `keyed_groups`: Groups instances based on a specific tag (e.g., `tags.Application`).
- `compose`: Defines how to compose the inventory, setting `ansible_host` to use the public DNS name.

### Ansible Configuration (ansible.cfg)

The `ansible.cfg` file is used to configure Ansible settings. Below is an example configuration that includes the path to the `aws_ec2.yml` inventory file:

```ini
[defaults]
interpreter_python = auto_silent
inventory = ./staging/aws_ec2.yml
roles_path = ./roles

[inventory]
enable_plugins = aws_ec2

[ssh_connection]
ssh_args = -o IdentityFile=./.ssh/id_rsa -o User=ec2-user
```

#### Explanation of Configuration:

- `[defaults]`: Sets general configurations, including the path to the inventory file and roles.
- `[inventory]`: Enables the `aws_ec2` inventory plugin.
- `[ssh_connection]`: Configures SSH connection options.

## Inventory Graph

After configuring the dynamic inventory, you can visualize the inventory graph using the following command:

```bash
ansible-inventory --graph
```

### Inventory Graph:

```plaintext
@all:
  |--@ungrouped:
  |--@aws_ec2:
  |  |--ec2-54-224-16-251.compute-1.amazonaws.com
  |  |--ec2-54-211-124-91.compute-1.amazonaws.com
  |  |--ec2-107-23-176-77.compute-1.amazonaws.com
  |--@wordpress:
  |  |--ec2-54-224-16-251.compute-1.amazonaws.com
  |--@prometheus:
  |  |--ec2-54-211-124-91.compute-1.amazonaws.com
  |--@docker:
  |  |--ec2-107-23-176-77.compute-1.amazonaws.com
```

The inventory graph illustrates the grouping of instances based on the specified tag (`Application`). Instances are grouped under `@wordpress`, `@prometheus`, and `@docker`.

## Conclusion

This documentation outlines the configuration of the `aws_ec2` dynamic inventory plugin in Ansible for AWS instances. With this setup, you can efficiently manage and automate tasks across your AWS infrastructure using Ansible. Ensure that your AWS credentials are properly configured to enable Ansible to interact with AWS services.
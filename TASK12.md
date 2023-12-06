# Configuring Prometheus for EC2 Service Discovery with IAM Role

In this documentation, we'll guide you through the process of configuring Prometheus to automatically discover and scrape EC2 instances using the EC2 Service Discovery feature. Additionally, IAM roles will be used to securely grant Prometheus the necessary permissions for discovering EC2 instances.

## Introduction

Prometheus, a powerful open-source monitoring and alerting toolkit, can be configured to dynamically discover and scrape EC2 instances in your AWS environment using EC2 Service Discovery. This feature simplifies the process of monitoring instances as they scale up or down. To enhance security, IAM roles will be employed to manage permissions.

## Prerequisites

Before proceeding, ensure you have completed the following prerequisites:

1. **Prometheus Installed:** Prometheus should be installed and running on an EC2 instance.
2. **IAM Role for EC2 Instances:** Create an IAM role with the necessary permissions to allow EC2 instances to be discovered by Prometheus.

## Step-by-Step Guide
Certainly! Here's the combined documentation with the updated Prometheus configuration script:

---

# Configuring Prometheus for EC2 Service Discovery with IAM Role

In this documentation, we'll guide you through the process of configuring Prometheus to automatically discover and scrape EC2 instances using the EC2 Service Discovery feature. Additionally, IAM roles will be used to securely grant Prometheus the necessary permissions for discovering EC2 instances.

## Introduction

Prometheus, a powerful open-source monitoring and alerting toolkit, can be configured to dynamically discover and scrape EC2 instances in your AWS environment using EC2 Service Discovery. This feature simplifies the process of monitoring instances as they scale up or down. To enhance security, IAM roles will be employed to manage permissions.

## Prerequisites

Before proceeding, ensure you have completed the following prerequisites:

1. **Prometheus Installed**: Prometheus should be installed and running on an EC2 instance.

2. **IAM Role for EC2 Instances**: Create an IAM role with the necessary permissions to allow EC2 instances to be discovered by Prometheus.

## Step-by-Step Guide

### 1. Create IAM Role for EC2 Instances

Create an IAM role with the required policies to enable EC2 instances to be discovered by Prometheus. The role should have the `AmazonEC2ReadOnlyAccess` policy attached.

#### Terraform Script

```hcl
resource "aws_iam_role" "prometheus" {
  name = "prometheus"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "prometheus_policy" {
  name        = "prometheus_policy"
  role        = aws_iam_role.prometheus.id

  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action   = "ec2:DescribeInstances",
      Effect   = "Allow",
      Resource = "*"
    }]
  })
}

resource "aws_iam_instance_profile" "prometheus" {
  name = "prometheus-2d2qd"
  role = aws_iam_role.prometheus.name
}
```

### 2. Attach IAM Role to EC2 Instances and Launch EC2 Instances Using Terraform Module

Now, attach the IAM role created in the previous step to the EC2 instances that Prometheus should discover. Additionally, use the Terraform module `prometheus_ec2` from task 6 to launch EC2 instances with the configured Prometheus settings.


   ```hcl
   module "prometheus_ec2" {
     # ... (previous configuration)
   
     iam_role  = aws_iam_instance_profile.prometheus.name
   
     # ... (remaining configuration)
   }
   ```

### 3. Apply Terraform Configuration

Apply the Terraform configuration to create the IAM role, associated policies, and instance profile.

```bash
terraform init
terraform apply
```

### 4. Update Prometheus Configuration

Modify the Prometheus configuration file (`prometheus.yml`) to include the EC2 service discovery configuration.

```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus_master'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    metrics_path: '/metrics'
    ec2_sd_configs:
    - region: {{ placement['region'] }}
      port: 9100
      filters:
        - name: tag:Name
          values:
          - docker*
          - wordpress*
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance

  - job_name: 'apache'
    ec2_sd_configs:
    - region: {{ placement['region'] }}
      port: 9117
      filters:
        - name: tag:Name
          values:
          - docker*
          - wordpress*
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance

  - job_name: 'docker_engine'
    ec2_sd_configs:
    - region: {{ placement['region'] }}
      port: 9323
      filters:
        - name: tag:Name
          values:
          - docker*
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance
```
The provided Prometheus configuration script defines jobs for scraping metrics from various sources. It utilizes EC2 Service Discovery to dynamically identify targets based on specific tags and ports within an AWS environment. The configuration includes jobs for monitoring Prometheus itself, as well as targets like node_exporter, Apache, and Docker engine exporters. These settings allow Prometheus to adaptively discover and monitor instances, providing flexibility and scalability for AWS-centric monitoring. Adjustments to the script can be made to suit specific environment requirements.

### 5. Apply Terraform Configuration

Now, apply the Terraform configuration to launch the EC2 instances.

```bash
terraform init
terraform apply
```

### 6. Verify Service Discovery

Monitor Prometheus logs or use

 the Prometheus web interface to ensure that EC2 instances launched by the Terraform module are being discovered and scraped.

- Prometheus Web Interface: http://your-prometheus-ip:9090/targets

### 7. Troubleshooting

If you encounter issues, check the Prometheus logs and verify that the IAM role permissions, EC2 instance tags, and other configurations are correctly set in both the Terraform module and Prometheus configuration.

## Conclusion

Congratulations! You have successfully configured Prometheus for EC2 service discovery using IAM roles and automated the deployment of EC2 instances with Terraform. This setup ensures secure and automated discovery of EC2 instances, allowing Prometheus to effectively monitor and scrape metrics from your AWS environment.

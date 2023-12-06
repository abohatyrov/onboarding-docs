## 2. Packer Configuration

In this section, we'll delve into the details of the Packer configuration files (`docker.json`, `wordpress.json`, `prometheus.json`). Each configuration file defines how Packer builds your Amazon Machine Image (AMI) and includes details about the base image, instance type, region, and provisioning scripts.

### a. Docker Configuration (`docker.json`)

```json
{
  "variables": {
    "ami_name_prefix": "dk"
  },
  "builders": [{
    "ami_description": "An AMI with Docker, based on RHEL.",
    "ami_name": "{{user `ami_name_prefix`}}-{{isotime | clean_resource_name}}",
    "instance_type": "t2.micro",
    "region": "{{user `region`}}",
    "source_ami_filter": {
      "filters": {
        "architecture": "x86_64",
        "block-device-mapping.volume-type": "gp2",
        "name": "RHEL-9.3.0_HVM-20231101-x86_64-5-Hourly2-GP2",
        "root-device-type": "ebs",
        "virtualization-type": "hvm"
      },
      "most_recent": true,
      "owners": ["309956199498"]
    },
    "ssh_username": "ec2-user",
    "type": "amazon-ebs"
  }],
  "provisioners": [{
    "inline": [
      "echo 'Sleeping for 30 seconds to give RHEL enough time to initialize (otherwise, packages may fail to install).'",
      "sleep 30",
      "sudo dnf update -y",
      "sudo dnf install -y yum-utils",
      "sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo",
      "sudo dnf install -y docker-ce"
    ],
    "type": "shell"
  }]
}
```

**Description:**
- `variables`: Defines variables used in the configuration. Here, `ami_name_prefix` is set to "dk".
- `builders`: Specifies the configuration for building the AMI, including instance type, region, and base image details.
- `source_ami_filter`: Filters the source image based on specified criteria.
- `provisioners`: Describes the provisioning steps, which include waiting for initialization, updating the system, and installing Docker.

### b. Wordpress Configuration (`wordpress.json`)

```json
{
  "variables": {
    "ami_name_prefix": "wp"
  },
  "builders": [{
    "ami_description": "An AMI with dependencies for Wordpress, based on RHEL.",
    "ami_name": "{{user `ami_name_prefix`}}-{{isotime | clean_resource_name}}",
    "instance_type": "t2.micro",
    "region": "{{user `region`}}",
    "source_ami_filter": {
      "filters": {
        "architecture": "x86_64",
        "block-device-mapping.volume-type": "gp2",
        "name": "RHEL-9.3.0_HVM-20231101-x86_64-5-Hourly2-GP2",
        "root-device-type": "ebs",
        "virtualization-type": "hvm"
      },
      "most_recent": true,
      "owners": ["309956199498"]
    },
    "ssh_username": "ec2-user",
    "type": "amazon-ebs"
  }],
  "provisioners": [{
    "inline": [
      "echo 'Sleeping for 30 seconds to give RHEL enough time to initialize (otherwise, packages may fail to install).'",
      "sleep 30",
      "sudo dnf update -y",
      "sudo dnf install -y mysql mysql-server python3-PyMySQL python3-firewall httpd wget httpd-devel php php-curl php-gd php-mbstring php-xml php-soap php-intl php-zip php-mysqlnd"
    ],
    "type": "shell"
  }]
}
```

**Description:**
- Similar to the Docker configuration but installs packages required for a Wordpress environment in the provisioning step.

### c. Prometheus Configuration (`prometheus.json`)

```json
{
  "variables": {
    "ami_name_prefix": "pr"
  },
  "builders": [{
    "ami_description": "An AMI with dependencies for Prometheus, based on RHEL.",
    "ami_name": "{{user `ami_name_prefix`}}-{{isotime | clean_resource_name}}",
    "instance_type": "t2.micro",
    "region": "{{user `region`}}",
    "source_ami_filter": {
      "filters": {
        "architecture": "x86_64",
        "block-device-mapping.volume-type": "gp2",
        "name": "RHEL-9.3.0_HVM-20231101-x86_64-5-Hourly2-GP2",
        "root-device-type": "ebs",
        "virtualization-type": "hvm"
      },
      "most_recent": true,
      "owners": ["309956199498"]
    },
    "ssh_username": "ec2-user",
    "type": "amazon-ebs"
  }],
  "provisioners": [{
    "inline": [
      "echo 'Sleeping for 30 seconds to give RHEL enough time to initialize (otherwise, packages may fail to install).'",
      "sleep 30",
      "sudo dnf update -y",
      "sudo dnf install -y jq"
    ],
    "type": "shell"
  }]
}
```

**Description:**
- Similar to the other configurations but installs the `jq` package, a lightweight JSON processor.

### 3. Build Docker AMI

Run the following command to build the Docker AMI:

```bash
packer build docker.json
```

### 4. Build Wordpress AMI

Run the following command to build the Wordpress AMI:

```bash
packer build wordpress.json
```

### 5. Build Prometheus AMI

Run the following command to build the Prometheus AMI:

```bash
packer build prometheus.json
```

### 6. Verify AMIs

After the build process completes, log in to your AWS Console and navigate to the EC2 Dashboard. Verify that the new AMIs with the specified prefixes are available in your chosen region.

## Conclusion

Congratulations! You've successfully created custom Amazon Machine Images with pre-installed packages using Packer. Feel free to customize the configurations further to suit your specific use cases.
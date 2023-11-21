# Launch a Docker Container with Apache Server using Terraform
We'll use Terraform to deploy a Docker container running an Apache server on the existing EC2 instance. In this documentation we will work with previous steps from Task 3.

# Introduction 
Docker is a containerization platform that simplifies the deployment and management of applications by encapsulating them in lightweight, portable containers. Containers are standalone, executable packages that include everything needed to run a piece of software, including the code, runtime, libraries, and system tools.

# Prerequisites
Before you begin, make sure you have the following prerequisites in place:

1. __Terraform Installation:__
Follow the official [Terraform installation guide](https://developer.hashicorp.com/terraform/install) to install Terraform on your machine.
2. __Docker Installation:__ Follow the official [Docker installation guide](https://docs.docker.com/engine/install/) to install Docker on your machine. You need to install it on localh machine where Terraform is hosting.

# Let's Get Started:
Proceed to the next sections to follow the step-by-step guide on launching a Docker container with Apache using Terraform.

## Step 1: Configure terraform
Firstly, let's see what directory structure we need to do this. We will create a module to launch Docker, so you need to create few files and directory. You can use following script to create necessary files:
```bash
# Create directories
mkdir -p modules/docker

# Create Terraform files
touch modules/docker/main.tf modules/docker/variables.tf modules/docker/providers.tf
```
After execution you'll get next structure

```bash
📦 Project Directory
 ┣ 📂.ssh
 ┣ 📂modules
 ┃ ┗ 📂ec2
 ┃ ┃ ┣ 📜main.tf
 ┃ ┃ ┣ 📜outputs.tf
 ┃ ┃ ┗ 📜variables.tf
 ┃ ┗ 📂docker
 ┃ ┃ ┣ 📜main.tf
 ┃ ┃ ┣ 📜providers.tf
 ┃ ┃ ┗ 📜variables.tf
 ┣ 📜main.tf
 ┣ 📜outputs.tf
 ┣ 📜providers.tf
 ┗ 📜variables.tf
```

You can see that it is a bit different from EC2 module. It s because we need to specify provider inside the module, so we can use it. And also we do not need outputs file here.
```bash
# module/docker/provider.tf
provider "docker" {}
```

But also we need to specify a provider and provider version in the root module. Do this, by opening the `providers.tf` file in the root of your project and paste next script:
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    # Add Docker provider source and version
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13"
    }
  }
}

provider "aws" {
  region = var.region

  shared_config_files      = [".aws/config"]
  shared_credentials_files = [".aws/credentials"]
  profile                  = "default"
}

# Use Docker provider
provider "docker" {}
```

In updated file you can see AWS provider from previous task and new Docker provider which in this case provided by `kreuzwerker`.

## Step 2: Create docker module
The Docker module may define resources such as Docker container and image.

```hcl
# modules/docker/main.tf
resource "docker_image" "this" {
  name         = "${var.image}:${var.tag}"
  keep_locally = false
}

resource "docker_container" "this" {
  name  = var.container_name
  image = docker_image.this.image_id

  ports {
    internal = var.internal_port
    external = var.external_port
  }
}
```
Here we creates image resource in which we use image with tag for our container and Docker container with configured port forwarding and image from previous resource. We need to specify variables for this module so we can use it properly.

## Step 3: Define Variables
If you don't know what is variables in Terraform modules you can read about that in Task 3 documentation. Now i will only show you how to specify them and what variables we need.

```hcl
# modules/docker/variables.tf
variable "image" {
  description = "The Docker image to deploy"
  default     = "httpd"
  type        = string
}

variable "tag" {
  description = "The tag of the Docker image to deploy"
  default     = "latest"
  type        = string
}

variable "container_name" {
  description = "The name of the Docker container"
  default     = "apache-container"
  type        = string
}

variable "internal_port" {
  description = "The internal port of the Docker container"
  default     = 80
  type        = number
}

variable "external_port" {
  description = "The external port of the Docker container"
  default     = 8080
  type        = number
}
```

## Step 4: Define Module in the Root
The last one step before applying terraform is to define docker module in the root files, so we can use it. Firstly, lets specify module in `main.tf`.

```hcl
# main.tf
module "apache_container" {
  source = "./modules/docker"

  image          = "httpd"
  tag            = "latest"
  container_name = "apache-container"
  internal_port  = 80
  external_port  = 8080
}
```

Specify here image name, image tag, container name that dcker will assign to it, and ports. Internal port is a port inside the container, and external port is a port on your machine where Docker Engine is hosting.

## Step 4: Innitialize and Apply Terraform
Initialize the Terraform project:
```bash
terraform init
```
Apply the Terraform configuration:
```bash
terraform apply
```
Confirm by typing yes when prompted. This will create a Docker container with Apache Web Server running in it.

If you done with you project and you don't need this infrastructure anymore you can clean up it by running following command:
```bash
terraform destroy
```
Confirm by typing yes when prompted.

# Conclusion
To check if Apache is working open your web browser and navigate to the IP address or domain name of your target server.
```
http://docker_server_ip:8080
```
Replace `docker_server_ip` with the actual IP address or domain of your server.

Congratulations! You have successfully deployed Docker container with Apache web server.
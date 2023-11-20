You need also to specify Docker provider in the docker module. It is necessary for docker module to work.
```bash
# module/docker/provider.tf
provider "docker" {}
```

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
The next step is to specify variables 

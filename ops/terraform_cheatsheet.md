# Terraform

## Installation

- download terraform [here](https://www.terraform.io/downloads.html)
- unzip and move it to `PATH`

## Use-cases

### 1 - Apply your first terraform script

- Create new directory
- Create `main.tf` file
- Your scripts depends highly on infrastructure that you're trying to deploy, for example using AWS we need to specify the provider as `aws` like so:

```
provider "aws" {
  access_key = "YOUR_ACCESS_KEY"
  secret_key = "YOUR_SECRET_KEY"
  region     = "YOUR_PREFERRED_REGION"
}
```

- Then, you can specify your resources, for example:

```
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"
}
```

the first argument is the type of resource and the second argument is its name.

> if on AWS, get aws access and secret [here](https://console.aws.amazon.com/iam/home?#security_credential)

- Lastly, you can initialize, show, plan and apply using these commands:

```
terraform init
terraform show
terraform plan
terraform apply
```

### 2 - Changes on Infrastructure

In case of changes, we can always modify our `*.tf` files and use these commands to plan and apply:

```
terraform plan
terraform apply
```

### 3 - Destroy Infrastructure

If you want to destroy everything specified on your script, you can use `terraform destroy` command.

### 4 - Implicit and Explicit Dependencies

- Implicit dependencies via interpolation expressions are the primary way to inform Terraform about these relationships, and should be used whenever possible.

```
resource "aws_eip" "ip" {
  instance = "${aws_instance.example.id}"
}
```

- Explicit dependencies on the other hand, have to be stated using `depends_on`.

```
resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  # Tells Terraform that this EC2 instance must be created only after the
  # S3 bucket has been created.
  depends_on = ["aws_s3_bucket.example"]
}
```

### 5 - Provisioning

If we use image-based infrastructure, what we have learned so far is good enough. However in cases where we are not using that (normal provisioning) we may have to run some scripts, we can use `provisioner` block for that purpose:

```
resource "aws_instance" "example" {
  ami           = "ami-b374d5a5"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo ${aws_instance.example.public_ip} > ip_address.txt"
  }
}
```

multiple provisioner block can be added, TF also support many provisioners other than `local-exec` see [here](https://www.terraform.io/docs/provisioners/index.html) for details.

**TF Characteristics on Provisioning**

If a resource successfully creates but fails during provisioning, Terraform will error and mark the resource as "tainted." A resource that is tainted has been physically created, but can't be considered safe to use since provisioning failed.

When you generate your next execution plan, Terraform will not attempt to restart provisioning on the same resource because it isn't guaranteed to be safe. Instead, Terraform will remove any tainted resources and create new resources, attempting to provision them again after creation.

Provisioners can also be defined so that run only occurs during a destroy operation. These are useful for performing system cleanup, extracting data, etc.

### 6 - Input Variables

We can also define variables and utilize it within our scripts. TF has specific way on knowing which variables to use during execution.

- Defining our variables, we can also specify defaults

```
variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "us-east-1"
}
```

- To utilize it, we can do this

```
provider "aws" {
  access_key = "${var.access_key}"
  secret_key = "${var.secret_key}"
  region     = "${var.region}"
}
```

These are the sequences that TF used to infer variables from most-prioritized to least-prioritized:

- Command-line flags
- From a file
- From env vars
- UI input
- Variable defaults

These are other non-primitive types that TF understand.

- Lists
- Maps

### 7 - Output Variables

Outputs are a way to tell TF what data is important. This data is outputted when apply is called, and can be queried using the terraform output command.

Let's define an output to show us the public IP address of the elastic IP address that we create. Add this to any of your `*.tf` files:

```
output "ip" {
  value = "${aws_eip.ip.public_ip}"
}
```

`terraform apply` highlights the outputs. You can also query the outputs after apply-time using terraform output:

```
$ terraform output ip
50.17.232.209
```

### 7 - Modules

Modules in Terraform are self-contained packages of Terraform configurations that are managed as a group. Modules are used to:

- Create reusable components
- Improve organization
- Treat pieces of infrastructure as a black box

The `Terraform Registry` includes a directory of ready-to-use modules for various common purposes, which can serve as larger building-blocks for your infrastructure.

In this example, we're going to use the Consul Terraform module for AWS, which will set up a complete Consul cluster. This and other modules can found via the search feature on the Terraform Registry site.

```
provider "aws" {
  access_key = "AWS ACCESS KEY"
  secret_key = "AWS SECRET KEY"
  region     = "us-east-1"
}

module "consul" {
  source = "hashicorp/consul/aws"

  aws_region  = "us-east-1" # should match provider region
  num_servers = "3"
}
```

The source attribute is the only mandatory argument for modules. It tells Terraform where the module can be retrieved. Terraform automatically downloads and manages modules for you.

Just as the module instance had input arguments such as num_servers above, module can also produce output values, similar to resource attributes.

```
output "consul_server_asg_name" {
  value = "${module.consul.asg_name_servers}"
}
```

### 9 - Remote Backend using Consul

First, we'll use Consul as our backend. Consul is a free and open source solution that provides state storage, locking, and environments. It is a great way to get started with Terraform backends.

```
terraform {
  backend "consul" {
    address = "demo.consul.io"
    path    = "getting-started-RANDOMSTRING"
    lock    = false
  }
}
```

## Next steps

- https://www.terraform.io/docs/index.html
- https://www.terraform.io/intro/examples/index.html
- https://www.terraform.io/docs/import/index.html

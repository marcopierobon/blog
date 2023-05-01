---
template: blog-post
title: Code reuse in Terraform part 1/6
slug: terraform-reuse-1
date: 2023-05-01 21:00
description: How to reuse code in terraform
---
**Introduction**

Code reuse is a software engineering practice that enables the reuse of existing code to perform similar tasks. In the world of Infrastructure as Code (IaC), code reuse is a highly desirable practice that promotes efficiency, consistency, and maintainability. Terraform, enables developers to create, manage, and update infrastructure resources through declarative configuration files. This series of articles explores various techniques and strategies for maximizing code reuse in Terraform and explores their advantages and disadvantages.

## Terraform 101

### Terraform Configuration Files

Terraform configuration files are written in HashiCorp Configuration Language (HCL) and have a .tf extension. These files define the infrastructure resources, their properties, and dependencies. Configuration files can include variables, data sources, and output values.

Here's an example of a configuration file that creates a VM in AWS:

```terraform
# Define an AWS EC2 instance
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "example-instance"
  }
}
```

### Terraform Providers

Providers are plugins that allow Terraform to interact with various infrastructure platforms, such as AWS, Azure, Google Cloud Platform, etc. Each provider exposes a set of resource types and data sources that can be used within Terraform configuration files.


Here's an example of an AWS provider being setup:

```terraform
provider "aws" {
  region = "us-west-2"
}
```

### Terraform State

Terraform maintains a state file that maps the resources defined in the configuration files to the infrastructure resources that are deployed. The state file is necessary for tracking changes, managing dependencies, and enabling collaboration among team members.

Terraform supports locking at the state file level using a number of different mechanisms, such as a DynamoDB table or a Consul cluster. These mechanisms ensure that only one user or tool can edit the state file at a time, preventing conflicts and ensuring that the infrastructure can be managed reliably.


Here's an example of a state file after having deployed a VM in AWS:

```terraform
{
  "version": 4,
  "terraform_version": "0.14.9",
  "serial": 1,
  "lineage": "1a2b3c4d-5e6f-7g8h-9i10-j1k2l3m4n5o6",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "example-instance",
      "provider": "provider.aws",
      "instances": [
        {
          "attributes": {
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "tags": {
              "Name": "example-instance"
            },
            "id": "i-1234567890abcdefg",
            "private_ip": "10.0.0.1",
            "public_ip": "54.0.0.1",
            "subnet_id": "subnet-12345678"
          }
        }
      ]
    }
  ]
}
```


### Terraform Modules

Modules are reusable packages of Terraform configuration files that encapsulate a specific functionality. Modules can be composed of resources, data sources, and other modules. They can be versioned and shared through various channels, such as version control systems or module registries.

A simple module can look like a main.tf file like the one below, places in an example-instance folder:

```terraform
variable "ami" {
  description = "AMI for the EC2 instance"
}

variable "instance_type" {
  description = "Instance type for the EC2 instance"
}

resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    Name = "example-instance"
  }
}
```

And can be referenced by a terraform project, located one folder up from the example-instance mentioned before:

```terraform
module "example-instance" {
  source = "./example-module"

  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

After having set the stage, we're ready to go deep into code reuse in terraform. In the next article we'll talk about *Code Reuse Through Modularization*.

Stay tuned!
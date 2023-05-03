---
template: blog-post
title: Code reuse in Terraform part 4/6
slug: terraform-reuse-4
date: 2023-05-03 09:08
description: How to reuse code in terraform using Data Sources
---
## Code Reuse Through Data Sources

Using data sources enables you to fetch existing data and configuration from existing resources, either within your current project or from external systems. 
By using data sources, you can avoid duplicating configuration information and helps avoiding duplication and inconsistencies in your code.
Here are some techniques to achieve code reuse through data sources:

### 4.1. Remote State Data Source

The *terraform_remote_state* data source allows you to access the output values of another Terraform configuration managed within the same workspace or in a different one. 
This feature is also useful when you have separate configurations for different layers of your infrastructure and need to share information between them.

Example:

```
# In the network configuration
output "vpc_id" {
  value = aws_vpc.main.id
}

# In the application configuration
data "terraform_remote_state" "network" {
  backend = "remote"

  config = {
    organization = "my-org"
    workspaces = {
      name = "network-prod"
    }
  }
}

resource "aws_subnet" "app" {
  cidr_block = "10.0.1.0/24"
  vpc_id     = data.terraform_remote_state.network.outputs.vpc_id
}
```

### 4.2. Provider Data Sources

Many Terraform providers have data sources that allow you to query and retrieve information about existing resources managed via that provider. 
This allows you to reuse configuration data from external systems and prevent duplication in your code.

AWS Data Source Example:

```
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical (Ubuntu) owner ID
}

resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

Azure Data Source Example:

```
data "azurerm_resource_group" "example" {
  name = "my-resource-group"
}

resource "azurerm_virtual_network" "example" {
  name                = "example-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = data.azurerm_resource_group.example.location
  resource_group_name = data.azurerm_resource_group.example.name
}
```

By using data sources, you can significantly reduce duplication in your code and ensure that your configurations remain consistent across different repositories. This approach promotes reusability, simplifies maintenance, and increases the  robustness of your infrastructure code.

In the next article we'll talk about *Code Reuse Leveraging Existing Terraform Modules*.

Stay tuned!
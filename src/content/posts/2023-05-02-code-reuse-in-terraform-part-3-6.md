---
template: blog-post
title: Code reuse in Terraform part 3/6
slug: terraform-reuse-3
date: 2023-05-02 13:06
description: How to reuse code in terraform using the DRY principle
---
## Code reuse using the DRY principle

The DRY (Don't Repeat Yourself) principle is a software development principle that emphasizes the importance of avoiding duplication in code and logic. 
The idea is to write code that is reusable, modular, and maintainable. The DRY principle can be very useful when working with Terraform.

Here are some advantages of using the DRY principle in Terraform:

1. Reduce code duplication: By using the DRY principle, you can avoid writing the same code over and over again. This can save time and reduce the risk of introducing bugs into your codebase. Also, whenever an error is found it can be fixed on a single place.

2. Enhance modularity: With the DRY principle, you can break your Terraform code into smaller, more manageable modules. As this makes you reason about how to split each module, it fosters reusability.

3. Promote consistency: By using the DRY principle, you end up with less code to maintain, which makes it easy to ensure that your Terraform code is consistent across your infrastructure. This can help ensure that all of your resources are configured in the same way.

4. Improve readability: DRY code tends to be more readable than code that is duplicated across multiple files. This is due by providing abstractions that can then be reused in different codebases.

There are 3 main ways to implement the DRY principle in terraform: *Count and For_each*, *Local Values* and *Dynamic Blocks*.

### 3.1. Count and For_each

The count and for_each meta-arguments allow you to create multiple instances of a resource or module based on the items in a collection, such as a list or a map. 

count Example:

```
resource "aws_security_group" "example" {
  name        = "example-${count.index}"
  description = "Example security group ${count.index}"
  count       = 3
}
```

for_each Example:

```
variable "subnets" {
  default = {
    app = "10.0.1.0/24"
    db  = "10.0.2.0/24"
  }
}

resource "aws_subnet" "example" {
  for_each      = var.subnets
  cidr_block    = each.value
  vpc_id        = aws_vpc.main.id
  tags = {
    Name = each.key
  }
}
```

### 3.2. Local Values

Local values are named expressions that can be used to simplify complex expressions or consolidate repeated values. By using local values, you can minimize repetition and improve the readability of your Terraform configuration, as a value can be defined once and reused in many places within a codebase.

Example:

```
locals {
  common_tags = {
    Terraform   = "true"
    Environment = "prod"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-0c94855ba95b798c7"
  instance_type = "t2.micro"
  tags          = merge(local.common_tags, { Name = "example-instance" })
}
```

### 3.3. Dynamic Blocks

Dynamic blocks allow you to generate multiple nested configuration blocks based on a collection. 
This feature is useful for managing resources with a variable number of nested blocks, such as security group rules or network interfaces. 


Example:

```
variable "ingress_rules" {
  default = [
    {
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  ]
}

resource "aws_security_group" "example" {
  name        = "example"
  description = "Example security group"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

In the next article we'll talk about *Code Reuse Through Data Sources*.

Stay tuned!

- [Code reuse in Terraform - Intro](https://www.pierobon.net/terraform-reuse-1)
- [Code reuse through modularization](https://www.pierobon.net/terraform-reuse-2)
- Code reuse using the DRY principle
- [Code Reuse Through Data Sources](https://www.pierobon.net/terraform-reuse-4)
- [Code Reuse Through public Terraform Modules](https://www.pierobon.net/terraform-reuse-5)
- [Best Practices for Terraform Code Reuse](https://www.pierobon.net/terraform-reuse-6)
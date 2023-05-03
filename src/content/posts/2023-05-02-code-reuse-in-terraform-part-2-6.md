---
template: blog-post
title: Code reuse in Terraform part 2/6
slug: terraform-reuse-2
date: 2023-05-02 09:57
description: How to reuse code in terraform through modularization
---
## Code reuse through modularization

Code reuse through modularization is an important technique in Terraform that can help in infrastructure management. Here are some of the key benefits:

1. Improved efficiency: Modularization can reduce the amount of code duplication in Terraform projects. By creating reusable modules for common infrastructure components, developers can save time and effort by not having to recreate the same resources over and over.

2. Consistency: Reusing modules across multiple projects can ensure that infrastructure is deployed consistently and according to the organisation's best practices. This can reduce the risk of errors and misconfigurations that can cause downtime or security issues.

3. Abstraction: By using modularization, Terraform projects and teams can be scaled up more easily. Adding new infrastructure components to a project becomes simpler when modules are used, reducing the time and effort required to manage large and complex infrastructure, as low level details are abstracted away from the day to day operations.

4. Maintenance: Modularization can make it easier to maintain Terraform projects over time. Changes to infrastructure components can be made more easily when they are defined in a reusable and self contained module, reducing the time and effort required to make updates. Also, changes become easier to validate, reducing the impact on existing infrastructure.

5. Collaboration: By using reusable modules, multiple teams can work together more easily on Terraform projects. Common infrastructure components can be defined once and used across multiple projects, reducing the amount of coordination required between teams. When properly implemented, this will allow teams to communicate via the terraform input variables, akin to an API first approach in development teams.

### 2.1. Creating Modules

To create a module, we need to organize the Terraform configuration files into a dedicated directory. 

This directory should contain:
- a main.tf file, which defines the module's resources
- a variables.tf file for input variables
- an outputs.tf file for output values. 

We can use the module block in the parent configuration to call the module and pass the required input variables.

Example:


```terraform
module "network" {
  source   = "./modules/network"
  vpc_cidr = "10.0.0.0/16"
}
```

### 2.2. Module Versioning

Version control systems, like Git, provide an effective way of versioning Terraform modules. 

By tagging specific commits, we can reference different versions of a module in out configuration files. 

This approach ensures that futures updates to a module do not break code depending on them.

In the example below, we are refencing the tag *v1.0.0* in the *network* repo.

```terraform
module "network" {
  source   = "git::https://example.com/modules/network.git?ref=v1.0.0"
  vpc_cidr = "10.0.0.0/16"
}
```

### 2.3. Module Hierarchy and Composition

Modules can be composed of other modules, allowing us to create a hierarchy of reusable components. 

This approach helps breaking down complex infrastructure into manageable and reusable pieces. 

For example, you can create a base module for a network and then extend it with specific configurations for different environments or applications.

Example:

```terraform
module "network" {
  source   = "./modules/network"
  vpc_cidr = "10.0.0.0/16"
}

module "app_subnet" {
  source   = "./modules/subnet"
  vpc_id   = module.network.vpc_id
  cidr     = "10.0.1.0/24"
}

module "db_subnet" {
  source   = "./modules/subnet"
  vpc_id   = module.network.vpc_id
  cidr     = "10.0.2.0/24"
}
```


### 2.4. Module Inputs and Outputs

By exposing input variables and output values, modules can be customized and extended without modifying the module's internal code. 

- Input variables define the parameters needed to configure the module
- Output values expose the resulting resources or data to the parent configuration.

For example in the modules/network/variables.tf we can define a vpc_cidr variable as an input parameter:

```terraform
variable "vpc_cidr" {
  type        = string
  description = "CIDR block for the VPC"
}
```

This variable will need to be provided when the module is referenced:

```terraform
module "network" {
  source   = "./modules/network"
  vpc_cidr = "10.0.0.0/16"
}
```


For example in the modules/network/outputs.tf we can define a vpc_id variable as an output value:

```terraform
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "The ID of the VPC"
}
```

This value will then be accessible by the calling module via the syntax `module.<MODULE NAME>.<OUTPUT NAME>`, in this case `module.network.vpc_id`.

In the next article we'll talk about *Using DRY for Code Reuse*.

Stay tuned!

- [Code reuse in Terraform - Intro](https://www.pierobon.net/terraform-reuse-1)
- Code reuse through modularization
- [Code reuse using the DRY principle](https://www.pierobon.net/terraform-reuse-3)
- [Code Reuse Through Data Sources](https://www.pierobon.net/terraform-reuse-4)
- [Code Reuse Through public Terraform Modules](https://www.pierobon.net/terraform-reuse-5)
- [Best Practices for Terraform Code Reuse](https://www.pierobon.net/terraform-reuse-6)
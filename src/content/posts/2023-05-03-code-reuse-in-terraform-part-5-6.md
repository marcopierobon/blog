---
template: blog-post
title: Code reuse in Terraform part 5/6
slug: terraform-reuse-5
date: 2023-05-03 19:23
description: How to reuse code in terraform through public Terraform Modules
---
## Code Reuse Through public Terraform Modules


Leveraging existing Terraform modules is a powerful way to achieve code reuse.

In this article, we will discuss the benefits of using public Terraform modules, how to find and include them into your projects, and provide an example of using a popular Terraform module from the Terraform Registry.

### 5.1 Benefits of Leveraging Existing Terraform Modules

Using existing Terraform modules has several advantages:

- it saves time and effort: Utilizing pre-built solutions can significantly reduce the time and effort required to create and manage your infrastructure, allowing you to focus on aspects that are specific to your project.

- it promotes best practices: Many existing Terraform modules are developed and maintained by experts in the field, ensuring that they follow best practices and are optimized for performance, security, and maintainability.

- it encourages consistency: Reusing modules across different projects or teams helps maintain consistency in your infrastructure and makes it easier to manage resources across various environments.

- it improve maintainability: By incorporating well-documented and tested modules, you can reduce the likelihood of bugs and simplify the process of updating your infrastructure as requirements change.

### 5.2 Finding and Incorporating Existing Terraform Modules

There are several resources available for finding existing Terraform modules:

- Terraform Registry (https://registry.terraform.io/): The official Terraform Registry is a comprehensive repository of Terraform modules and providers, developed and maintained by both the Terraform community and HashiCorp. You can search for modules by keyword, filter by provider, and sort by popularity or recent updates.

- GitHub: Many Terraform modules are hosted on GitHub, often within the repositories of organizations or individuals. You can search GitHub for Terraform modules using keywords and filters or explore popular repositories and users.

- Community resources: Blogs, forums, and social media platforms can be useful resources for discovering popular or highly recommended Terraform modules. You can also ask for recommendations from colleagues or online communities.

To incorporate an existing Terraform module into your project, follow these steps:
1. Locate the module in the Terraform Registry, GitHub, or other source.

2. Review the module's documentation to understand its inputs, outputs, and any dependencies or prerequisites.

3. Add a module block to your Terraform configuration file, specifying the module's source, version (if applicable), and any required input variables.

4. Run terraform init to fetch and initialize the module.

5. Use the module's outputs in your Terraform configuration as needed.

### 5.3 Example: Using the AWS VPC Module from the Terraform Registry

In this example, we will show how to incorporate the popular terraform-aws-modules/vpc/aws module from the Terraform Registry to create a VPC in AWS.

1. Go to the module's page on the Terraform Registry (https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) and review its documentation.

2. In your Terraform configuration file, add a module block to use the AWS VPC module:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
    public_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

    enable_nat_gateway = true
    single_nat_gateway = true

    tags = {
        Terraform = "true"
        Environment = "dev"
    }
}
```


3. Customize the input variables as needed for your specific requirements. In this example, we have defined a VPC with a CIDR block of `10.0.0.0/16`, three private subnets, and three public subnets distributed across three availability zones. We have also enabled a NAT gateway for the VPC.

4. Run `terraform init` to fetch and initialize the module.

5. Run `terraform apply` to deploy the resources.

6. If needed, you can use the module's outputs in your Terraform configuration. For example, to create an EC2 instance in one of the private subnets:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 LTS AMI
  instance_type = "t2.micro"

  subnet_id = module.vpc.private_subnets[0]

  tags = {
    Name = "example-instance"
  }
}
```

After having set the stage, we're ready to go deep into code reuse in terraform. In the next article we'll talk about *Best Practices for Terraform Code Reuse*.

Stay tuned!

- [Code reuse in Terraform - Intro](https://www.pierobon.net/terraform-reuse-1)
- [Code reuse through modularization](https://www.pierobon.net/terraform-reuse-2)
- [Code reuse using the DRY principle](https://www.pierobon.net/terraform-reuse-3)
- [Code Reuse Through Data Sources](https://www.pierobon.net/terraform-reuse-4)
- Code Reuse Through public Terraform Modules
- [Best Practices for Terraform Code Reuse](https://www.pierobon.net/terraform-reuse-6)


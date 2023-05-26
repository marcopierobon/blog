---
template: blog-post
title: Best practices for structuring a terraform repository
slug: /terraform-repo-structure
date: 2023-05-26 14:49
description: Best practices for structuring a new terraform repository
  minimising code duplication
featuredImage: /assets/terraform.png
---
## Structuring a Terraform Repository: Efficient Management of Multiple Environments

We  will explore how to structure a Terraform repository for managing multiple environments, in this case a development (dev) and production (prod) environment. Each environment will have an S3, EC2, and Lambda module and a dedicated state file stored in an S3 bucket. 

The principle guiding our approach is minimizing code duplication. The actual Terraform code will reside solely in the prod folder, with symbolic links (symlinks) created in the dev folder. Variables will be used to apply the appropriate values for each environment, leading to a clean and manageable repository.

### Repository Structure

Our Terraform repository will adhere to the following structure:

```
terraform-repo
│
└───dev
│   │   main.tf (symlink)
│   │   variables.tf (symlink)
│   │   backend.tf (symlink)
│   │   terraform.tfvars
│   │   locals.tf (symlink)
│
└───prod
    │   main.tf
    │   variables.tf
    │   backend.tf
    │   terraform.tfvars
    │   locals.tf
```

Each environment folder (*dev* and *prod*) will contain 5 key files: *main.tf*, *variables.tf*, *backend.tf*, *terraform.tfvars* and *locals.tfvars*.

The *main.tf* file defines Terraform code to provision our infrastructure. The *variables.tf* file defines variables that we'll use in our Terraform code. The *terraform.tfvars* provides the values for those variables that are specific to our environment, and *backend.tf* configures the backend for storing our Terraform state. 

In the dev environment, *main.tf*, *backend.tf*, *terraform.tfvars* and *locals.tfvars* will be symbolic links to their counterparts in the prod environment. This setup facilitates infrastructure code management while permitting variable customization for each environment.

### Environment Variables

The variable definition will be done in a single place (*prod/variables.tf*), but as a variable will have different values for the dev and prod environments, two *variables.tfvars* files will be required.

The variable definition file *prod/variables.tf* will be linked via a symbolic link from the dev folder.

```hcl
variable "region" {
  description = "AWS region"
  default     = "us-west-2"
}

variable "availability_zone" {
  description = "The Availability Zone where resources will be created"
  default     = "us-west-2a"
}

variable "environment_name" {
  description = "The name of the environment (e.g., 'dev' or 'prod')"
  default     = "dev"
}

variable "team_name" {
  description = "The name of the team"
  default     = "PinkUnicorns"
}

variable "s3_bucket_name" {
  description = "Name of the S3 bucket"
}

variable "ec2_instance_name" {
  description = "Name of the EC2 instance"
}

variable "ec2_instance_type" {
  description = "Instance type of the EC2 instance"
}

variable "parameter_store_db_name" {
  description = "The name of the parameter store for the DB name"
}

variable "parameter_store_db_user" {
  description = "The name of the parameter store for the DB user"
}

variable "db_name" {
  description = "The name of the database"
}

variable "db_user" {
  description = "The username for the database"
}
```

These are the values for the prod environment, that will be set in *prod/terraform.tfvars*:


```hcl
region                  = "us-east-1"
availability_zone       = "us-east-1"
environment_name        = "dev"
team_name               = "PinkUnicorns"
s3_bucket_name          = "my-dev-bucket-tf-setup"
ec2_instance_name       = "dev-ec2-instance"
ec2_instance_type       = "t2.micro"
parameter_store_db_name = "/dev/db_name"
parameter_store_db_user = "/dev/db_user"
db_name                 = "dev_database"
db_user                 = "dev_user"
```

And in *dev/terraform.tfvars* we will set the values for dev:

```hcl
region                  = "us-west-2"
availability_zone       = "us-west-2a"
environment_name        = "prod"
team_name               = "PinkUnicorns"
s3_bucket_name          = "my-prod-bucket-tf-setup"
ec2_instance_name       = "prod-ec2-instance"
ec2_instance_type       = "t2.large"
parameter_store_db_name = "/prod/db_name"
parameter_store_db_user = "/prod/db_user"
db_name                 = "prod_database"
db_user                 = "prod_user"
```

### Locals

In *prod/locals.tf* we'll define expressions that can be reused, but with the variable values coming from the *terraform.tfvars* file.
This also has the extra benefit to allow us to use these values in variable blocks.

This file will be defined in *prod/locals.tf* and linked via a symbolic link from the dev folder.
```hcl
locals {
  team_name = "PinkUnicorns"

  common_tags = {
    Team        = local.team_name
    Environment = var.environment_name
  }
}
```

### Configuring the Terraform Backend

The Terraform S3 backend doesn't support interpolation syntax directly in the backend configuration block, hence we can't use variables directly in the backend configuration.

This lack of support is intentional due to the order of operations for Terraform. When Terraform initializes, it reads the backend configurations before anything else. 
For this very reason, the backend configuration can't depend on values from resources or data sources because those don't exist until after initialization.

That'w why we cannot use symlinks in this case.

The *prod/backend.tf* should look like this:

```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket-tf-setup"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```

And the *dev/backend.tf* should look like this:

```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state-bucket-tf-setup"
    key    = "dev/terraform.tfstate"
    region = "us-east-1"
  }
}
```


### Main.tf

The main.tf definition will be done in a single place (*prod/main.tf*), and it will be referenced via a symbolic link from the dev folder.

```hcl
provider "aws" {
  region = var.region
}

data "aws_ami" "ubuntu_latest" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["099720109477"] # Canonical
}

module "s3_bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "~> 2.0"

  bucket = var.s3_bucket_name
  tags   = local.common_tags
}

module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "~> 3.0"

  name          = var.ec2_instance_name
  instance_type = var.ec2_instance_type
  ami           = data.aws_ami.ubuntu_latest.id
  tags          = local.common_tags
}

resource "aws_ssm_parameter" "db_name" {
  name  = var.parameter_store_db_name
  type  = "String"
  value = var.db_name
  tags  = local.common_tags
}

resource "aws_ssm_parameter" "db_user" {
  name  = var.parameter_store_db_user
  type  = "String"
  value = var.db_user
  tags  = local.common_tags
}
```

### Creating Symbolic Links

On Linux or MacOS, you can create symbolic links using the ln -s command:

Execute these commands in the dev directory.

```bash
ln -s ../prod/main.tf main.tf
ln -s ../prod/variables.tf variables.tf
ln -s ../prod/backend.tf backend.tf
ln -s ../prod/locals.tf locals.tf
```


On Windows, you can create symbolic links with the mklink command:

```bash
mklink main.tf ..\prod
```

### Conclusion

With this setup, we can reduce the amount of code duplication to a minimum. Although at times there might be some differences between different environments, those should be few and far between.
To reduce the impact of this happening, it helps to separate the *main.tf* file into each module being referenced, so that only the ones that are different will require an explicit rewrite.

Also, whenever there's the need to create a new environment, we can just copy the folder with the sym links (dev in this case), remove its .terraform folder, update the *backend.tf*, run a 
`terraform init` and we're all set.

You can find the code used in this article [here](https://github.com/marcopierobon/terraform-repo-setup/tree/master).

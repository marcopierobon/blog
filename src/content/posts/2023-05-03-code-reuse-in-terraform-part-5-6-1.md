---
template: blog-post
title: Code reuse in Terraform part 5/6
slug: terraform-reuse-5
date: 2023-05-03 19:49
description: How to reuse code in terraform through public Terraform Modules
---
### Best Practices for Terraform Code Reuse



As your infrastructure grows, so does the complexity of managing it. Effective code reuse in Terraform can help you create more efficient and maintainable infrastructure. In this articles series, we have discussed the best practices for Terraform code reuse, including modularization, adhering to the DRY principle, using data sources, leveraging existing modules, and following coding standards.

Here's a recap.

### Modularization

Modularization is the process of breaking down your Terraform configuration into smaller, reusable pieces called modules. Each module should have a well-defined interace and be responsible for managing a specific set of resources. This approach encourages code reuse, reduces duplication, and promotes maintainability.

- Organize modules by function: Create modules that handle specific tasks, such as provisioning a VPC, deploying an EC2 instance, or configuring a security group. This makes it easier to reuse and maintain your code.

- Limit module complexity: Keep your modules simple and focused on a single responsibility. This will make them easier to understand, test, and reuse across various projects.

- Use module inputs and outputs: Define input variables and outputs for your modules to make them more flexible and customizable. This allows users to tailor the module to their specific needs without modifying the module's code.

You can read more about modularization [here](https://www.pierobon.net/terraform-reuse-2).

### DRY (Don't Repeat Yourself) Principle

The DRY principle shows the importance of avoiding duplication in your code. By adhering to this principle, you can reduce the risk of inconsistencies, simplify maintenance, and improve code quality.

- Centralize resource configuration: If you have multiple resources that share common configurations, consider creating a module to centralize this configuration, and then reference it in your other resources.

- Use variables and locals for shared values: Use variables for values that are used in multiple places throughout your Terraform configuration. This will make it easier to update and maintain your code.

- Utilize Terraform functions: Leverage Terraform functions such as `lookup`, `merge`, or `for_each` to reduce duplication and create more dynamic and reusable configurations.

You can read more about applying the dry principle [here](https://www.pierobon.net/terraform-reuse-3).

### Use Data Sources

Data sources allow you to fetch data from existing resources in your infrastructure or from external sources. Using data sources can help you avoid duplicating code and ensure consistency across your environment.

- Reference existing resources: Instead of hardcoding values or duplicating resource configurations, use data sources to reference existing resources and their properties.
- Fetch dynamic data: Use data sources to fetch dynamic data, such as instance IDs, subnet IDs, or availability zones. This helps you create more flexible and reusable Terraform configurations.

You can read more about using data sources [here](https://www.pierobon.net/terraform-reuse-4).

### Leverage Public Modules

You can leverage existing Terraform modules from sources like the Terraform Registry or GitHub to save time and effort, promote best practices, and encourage consistency across your infrastructure.

- Use well-maintained modules: Choose modules that are actively maintained, well-documented, and have a strong community support.

- Customize modules as needed: Utilize input variables and outputs to tailor existing modules to your specific requirements.

You can read more about leveraging public modules [here](https://www.pierobon.net/terraform-reuse-5).

### Follow Coding Standards

Adhering to consistent coding standards helps improve code readability, maintainability, and reusability.

- Use a descriptive naming conventions: Choose clear and descriptive names for your resources, modules, variables, and outputs. This makes your code easier to understand and maintain.

- Format your code: Use tools like terraform fmt or editor extensions to automatically format your Terraform code according to established best practices.

- Document your code: Add comments and documentation to your Terraform code to explain its purpose, inputs, outputs, and any dependencies or prerequisites.

- Test your code: infrastructure code containing tests is less likely to cause bugs. In addition, well written tests can also act as an enhanced documentation.

This was the last article of this series.

You can access each individual article here:
- [Code reuse in Terraform - Intro](https://www.pierobon.net/terraform-reuse-1)
- [Code reuse through modularization](https://www.pierobon.net/terraform-reuse-2)
- [Code reuse using the DRY principle](https://www.pierobon.net/terraform-reuse-3)
- [Code Reuse Through Data Sources](https://www.pierobon.net/terraform-reuse-4)
- [Code Reuse Through public Terraform Modules](https://www.pierobon.net/terraform-reuse-5)
- Best Practices for Terraform Code Reuse

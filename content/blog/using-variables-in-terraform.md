---
author: Liam
title: "Using variables in Terraform"
date: 2023-07-23
description: "Using variables in Terraform resources to improve re-usability"
tags: ["terraform", "iac", "infrastructure"]
thumbnail: img/terraform-vars.png
---

## Introduction

In a previous blog post I have introduced Terraform and its core principles. In short; Terraform provides a declarative approach to infrastructure provisioning, allowing you to define your desired state and automatically manage the necessary resources across multiple cloud providers. One of the key features that makes Terraform highly flexible and reusable is its support for variables. In this blog post, we will explore Terraform variables and how they can enhance your infrastructure management workflow.

## Understanding Terraform Variables

Variables in Terraform are placeholders that allow you to parameterise your configuration, making it more dynamic and customisable. With variables, you can avoid hardcoding values in your Terraform code and instead use placeholders that can be easily overridden or modified based on specific environments or requirements.

## Declaring Variables

In Terraform, you can declare variables within your configuration using the `variable` block. Variables can have different types such as string, number, boolean, list, or map. You can define default values for variables or make them optional, depending on your use case. This flexibility allows you to set defaults while also giving you the ability to override them when needed.

## Passing Variable Values

Once you've declared variables, you can pass values to them in multiple ways. You can provide values directly in your Terraform code by assigning them during variable interpolation. Alternatively, you can use variable files, environment variables, or even command-line flags to provide values at runtime. This versatility enables you to manage variables across different environments, making your infrastructure code reusable and adaptable.

## Using variables

As part of improving my IaC I have moved my `tf` files for my homelab resources from individual folders to individual files, and I'll cover this and using Terraform modules in the next post as part of refactoring my IaC. My first stage of refactoring is to utilise variables more throughout my Terraform code. I will use a `variables.tf` file and abstract variables from individual `tf` files where required. I found the [DigitalOcean guide](https://www.digitalocean.com/community/tutorials/how-to-improve-flexibility-using-terraform-variables-dependencies-and-conditionals)  guide and [this guide on spacelift.io](https://spacelift.io/blog/how-to-use-terraform-variables) useful to learn more about and see examples of variables in use. I was then able to apply this to my own project.

Firstly create a `variables.tf` file and add any required variables, for example I have created a variable for the VM template I want to use in Proxmox to create a resource from:

```terraform
variable "vm_template" {
	type = string
	description = "VM template to clone"
	default = "ubuntu-cloud-20.04"
}
```

Then in your resource `.tf` file you can define the variable to use by using the `var.` to access variables by their name:

```terraform
clone = var.vm_template
```

## Module Input Variables

Terraform modules, which are reusable components of infrastructure code, can also leverage variables. By defining input variables for modules, you can further enhance their flexibility. Module input variables allow you to pass values from the calling module to the module being used, enabling configuration customisation and reuse at a higher level. I'll be covering modules more extensively in my next post.

## Remote State and Output Variables

Variables aren't limited to just input values. Terraform also supports output variables, which allow you to expose certain values from your infrastructure configuration. Output variables can be useful for sharing information between different Terraform configurations or for providing valuable insights about your infrastructure state. For example, the output variable below will output an instance IP to the terminal.

```terraform
output "instance_ip_addr" {
  value       = aws_instance.server.private_ip
  description = "The private IP address of the instance."
}
```

## Benefits of Using Variables

Using variables in your Terraform code offers several advantages. Firstly, it promotes reusability, as variables allow you to create generic and adaptable configurations that can be easily applied to different environments or projects. Secondly, variables enhance maintainability by providing a centralized location for configuration values, making it easier to manage and update them. Additionally, variables enable collaboration by separating infrastructure code from environment-specific values, allowing different teams or individuals to work on different parts of the codebase without conflicts.

## Conclusion

Terraform variables allow you to create flexible and reusable infrastructure code that can be easily customised and adapted to various environments. By leveraging variables, you can enhance the scalability, maintainability, and collaboration aspects of your infrastructure management workflow.

In upcoming blog posts, I'll be covering Terraform modules and how I refactored by code to use them and migrating a local state to a remote provider.

## Resources

- [DigitalOcean guide](https://www.digitalocean.com/community/tutorials/how-to-improve-flexibility-using-terraform-variables-dependencies-and-conditionals)  
- [spacelift.io - using Terraform variables](https://spacelift.io/blog/how-to-use-terraform-variables)
---
layout: post
title: Enhancing Terraform with Terragrunt
description: Keeping DRY and Dynamic
image: ../assets/images/enhancing-terraform-with-terragrunt/terragrunt-terraform.png
---

As the technology world rapidly progresses today, it has become inevitable for infrastructure management to be more efficient and automated. Traditional manual infrastructure setups not only cause time loss but also increase the risk of errors. This is where managing infrastructure as code comes into play. With **IaC**, infrastructure management can be easily automated.

![IaC Tools](/assets/images/enhancing-terraform-with-terragrunt/stack.jpg)

We also know that many different tools are available to implement this approach. For example, solutions provided by cloud providers such as **CloudFormation** in **AWS**, **Azure Resource Manager** in **Azure**, and **Deployment Manager** in **GCP** stand out, while **cloud-agnostic** options like **Pulumi**, **Crossplane**, and **OpenTofu/Terraform** are also commonly preferred. While each tool has its own unique advantages, this diversity can sometimes create challenges. Learning different solutions, creating separate definitions for each environment, and constantly dealing with code duplication can be exhausting for IT developers (SRE/DevOps/Cloud Engineers).

**In fact**, our fundamental expectation is quite simple: To use a single tool that ensures infrastructure written once works consistently across different environments without code duplication and manages the entire process from a single code repository. Achieving this goal will simplify processes and help teams work faster and more effectively.

Among these tools, **OpenTofu/Terraform** is the most popular. However, the vanilla version of **OpenTofu/Terraform** unfortunately does not fully meet our expectations. But we can address this by using a small helper tool called **Terragrunt**. So, what is **Terragrunt**?

#### What's Terragrunt?

![IaC Tools](/assets/images/enhancing-terraform-with-terragrunt/terragrunt-terraform.png)

**Terragrunt** is a flexible orchestration tool designed to scale Infrastructure as Code (IaC) with **OpenTofu/Terraform**. We can actually see that **Terragrunt** is different from the tools we mentioned earlier through this definition. Instead of providing IaC itself, it is designed as an orchestrator for **OpenTofu/Terraform**. In this case, we can easily say that the primary goal of Terragrunt is to extend the capabilities of **OpenTofu/Terraform**. Let’s briefly summarize what additional features **Terragrunt** brings to **OpenTofu/Terraform**:

1. **DRY (Don’t Repeat Yourself) Principle**: It reduces code duplication by defining common configurations once and reusing them across different environments and modules.
2. **Remote State Management**: Simplifies the configuration and management of remote states, ensuring teams can collaborate effectively and maintain consistent infrastructure states.
3. **Dependency Management**: Manages dependencies between modules, ensuring operations are carried out in the correct order and according to dependencies.
4. **Before and After Hooks**: Allows you to run custom scripts or commands before or after running Terraform commands, providing flexibility in your infrastructure deployment process.
5. **Environment-Specific Configurations**: This makes it easier to create environment-specific configurations (such as release, test, and production) using HCL (HashiCorp Configuration Language) interpolation, ensuring consistent environments are maintained.

You can use the [documentation](https://terragrunt.gruntwork.io/docs/getting-started/install/) for setting up **Terragrunt**. Please remember that **OpenTofu/Terraform** needs to be installed in order to use **Terragrunt**.

---

#### Example Scenario

After briefly discussing **Terragrunt** and its features, we can apply it through a simple example. Let’s say we want to set up a **VPC** on a cloud platform (for example, **Google Cloud Platform**) for our *Test*, *Release*, and *Production* environments. In that case, we first need to design the VPC module by creating a `vpc` folder inside a `modules` folder.

```plaintext
└── modules/
    └── vpc/
        ├── main.tf
        ├── provider.tf
        └── variables.tf
```

If we take a brief look at the files contained in our VPC module:

- **main.tf**: This file includes the definitions of the resources (in this scenario, only the VPC) provided by the provider we are using.
- **provider.tf**: Here, we will define and configure the `google` provider for the **Google Cloud Platform**.
- **variables.tf**: In this file, we will define the variables to be used in both `main.tf` and `provider.tf`.

We can start by designing the `provider.tf` file for our module. In this scenario, we mentioned that we would be using **Google Cloud Platform**. Therefore, we need to define the `google` provider here.

```json
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "6.14.1"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}
```

In `main.tf`, let’s define the necessary configurations for the **VPC** by utilizing the resources provided by this provider.

```json
resource "google_compute_network" "vpc_network" {
  name                    = var.network_name
  auto_create_subnetworks = false
}

resource "google_compute_firewall" "default" {
  name    = "${var.env}-gke-firewall"
  network = google_compute_network.vpc_network.name
  source_tags = ["medium-article-demo"]  
  allow {
    protocol = "icmp"
  }
  allow {
    protocol = "tcp"
    ports    = ["80", "443"]
  }
}

resource "google_compute_subnetwork" "subnetwork" {
  name          = var.subnetwork_name
  ip_cidr_range = var.subnetwork_cidr
  region        = var.region
  network       = google_compute_network.vpc_network.id
}
```

After creating the `provider.tf` and `main.tf` files, we realize some attributes have undefined values. To assign values to these attributes, we used variables. However, for **Terraform** to recognize these variables, we first need to define them in a file. For this purpose, we should create the `variables.tf` file and define these variables in it.

```json
variable "network_name" {}
variable "subnetwork_name" {}
variable "subnetwork_cidr" {}
variable "region" {}
variable "project_id" {}
variable "env" {}
```

Thus, we have created the template for our VPCmodule. You can further expand and customize this VPCmodule if desired. Now it’s time to incorporate **Terragrunt** into this scenario. We mentioned that we want to use this template across three different environments in our scenario. In that case, we should outline the following template for this scenario.

```plaintext
└── environments/
    ├── test/
    │   └── vpc/
    │       └── terragrunt.hcl
    ├── release/
    │   └── vpc/
    │       └── terragrunt.hcl
    └── production/
        └── vpc/
            └── terragrunt.hcl
```

**Terragrunt** is used with files that have an HCL extension. Since we will use three different environments in this scenario, each environment may have different `VPC` configurations. Therefore, we need to create separate `terragrunt.hcl` files for each environment, where we can define the corresponding values for the variables assigned to the attributes.

```json
terraform {
  source = "<your-vpc-module-path>"
}

inputs = {
  network_name    = "medium-article-dev-vpc"
  subnetwork_name = "medium-article-dev-subnet"
  subnetwork_cidr = "10.1.0.0/16"
  region = "us-west1"
  zone = "us-west1-b"
  project_id = "your-google-project-id"
  env = "development"
}
```

By updating these configurations for each environment, we can now create VPCs in our desired regions for projects within the GCP environment.

The **CLI** commands for **Terraform** and **Terragrunt** are similar. However, you cannot use **Terraform CLI** commands with **Terragrunt**. Therefore, to use Terragrunt, you need to initiate processes using its own CLI commands. In the directory corresponding to the environment where you want to operate (e.g., **environments/development/vpc**), you can run the following commands to use **Terragrunt**.

```bash
terragrunt plan
terragrunt apply
```

If you want to run it without navigating to the relevant directory, you can use it as shown below.

```bash
terragrunt run-all plan --terragrunt-include-dir .\environments\development\vpc
terragrunt run-all apply --terragrunt-include-dir .\environments\development\vpc
```

**Bonus:** If you want to run the VPCs for all environments at once without navigating to the relevant directories, you can use the following command.

```bash
terragrunt run-all plan --terragrunt-include-dir .\environments\**\vpc
terragrunt run-all apply --terragrunt-include-dir .\environments\**\vpc
```

#### Conclusion

To sum up, managing infrastructure as code has become essential for driving automation and minimizing human errors in modern IT environments. Many tools have been developed and are used both by cloud providers and third-party providers. However, learning and developing so many tools often results in additional costs for **DevOps/SRE** developers who push these processes forward. At this point, the use of orchestrator tools that can enhance the capabilities of a de facto product is crucial from a cost management perspective. **By adopting the DRY code principle, managing dependencies, and ensuring consistent environments, Terragrunt enables OpenTofu/Terraform to be used more effectively and dynamically.** As infrastructure complexity increases, using orchestration tools like **Terragrunt** will be a critical factor in simplifying management processes and ensuring the long-term success of IT teams.

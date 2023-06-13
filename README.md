# Local setup
## Prerequisites
- [Terraform](https://www.terraform.io/downloads.html)
- [IDE](https://www.jetbrains.com/idea/): IntelliJ IDEA
- [AWS CLI](https://aws.amazon.com/cli/): As part of installing this, you will need to make an AWS account if you don't have one and [configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) `awscli` to use that account.


# Terraform overview

Terraform is an open-source tool developed by HashiCorp that specializes in **provisioning infrastructure**. With Terraform, we can simply write code and execute a few simple CLI statements to create the **infrastructure** we need.

![Terraform diagram](https://www.devopsschool.com/blog/wp-content/uploads/2021/07/terraform-architecture-components-workflow-2.png)

The Terraform file will have the format “.tf” written in the HCL (Hashicorp Configuration Language) language. 
A typical Root Module will have three main Terraform files:
- **main.tf** including components: resources, provider, data source.
- **variables.tf** is where variables are declared.
- **outputs.tf** export value after terraform apply has finished executing.

Besides
- **terraform.tfstate** (State management, save file in  local or s3)
- **terraform.tfstate.backup** (Backup state management)


## Terraform Components
#### Providers

Providers are plugins that implement the resource types. They contain all the necessary code to authenticate and connect to a specific service, usually from a public cloud provider, on the user’s behalf. Here, we have the AWS provider to enable us to create AWS infrastructure.
```terraform
# main.tf

provider "aws" {
  region = "ap-northeast-1"
}
```

#### Input Variables

Input variables act as parameters, giving a way for users to customize a deployment without altering the configuration code. This also enables users to share configurations with others more easily. Variables are defined in `variables.tf` and can be overridden in `values.tfvars`.

```terraform
# variables.tf

variable "instance_name" {
  description = "Value of the Name tag for the EC2 instance"
  type        = string
  default     = "ExampleAppServerInstance"
}
```
```terraform
# value.tfvars

instance_name='server'

```
```bash
# Command use value.tfvars file

terraform plan -var-file='values.tfvars'
```

#### Data Sources

Data sources fetch and compute data for use in Terraform's configuration.
A data source is accessed via a special kind of resource known as a data resource, declared using a data block:
```terraform
# main.tf

data "aws_ami" "example" {
  most_recent = true

  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```
#### Resources
Resources are the most important element in the Terraform language. Resources describe one or more infrastructure objects. 
```terraform
# main.tf

resource "aws_instance" "server" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}

```
Each of our resources will have **input arguments** and **output attributes** values ​​depending on the resource type. The attributes may also include computed attributes, which are attributes that can only be determined once the resource has been created.
![Terraform resource](https://images.viblo.asia/97b4c7a4-44fc-4dea-aed1-1d0f1d102519.png)

#### Output Values
Output values are the equivalent of return values. We use them here to print relevant information about the deployment once it is deployed, such as the AMI ID we used and the Amazon Resource Name (ARN) of our EC2 instance.
Output values are defined for this folder in `outputs.tf`. Every data source and resource block also has outputs.
```
# outputs.tf

output "instance_ip_addr" {
  value = aws_instance.server.private_ip
}
```

### State
Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures.

This state is stored by default in a local file named **terraform.tfstate**, but we recommend storing it in Terraform Cloud to version, encrypt, and securely share it with your team.
## Terraform's provisioning workflow
![Terraform workflow](https://olohmann.github.io/azure-hands-on-labs/labs/07_iac/media/terraformcycle.png)
`init` - Initialize the (local) Terraform environment. Usually executed only once per session.
`plan` - Plan. Compare the Terraform state with the as-is state in the cloud, build and display an execution plan. This does not change change the deployment (read-only).
`apply` - Apply the plan from the plan phase. This potentially changes the deployment (read and write).
`destroy` - Destroy all resources that are governed by this specific terraform environment.
```bash
# command

terraform init
terraform plan
terraform apply
terraform destroy
```


# Next deployment
## The steps we take are as follows:
### Step 1: Use S3 to host `terraform.tfstate`
Note: (name of s3 bucket)
   - Staging: `aime-terraform-staging`
   - Prod: `aime-terraform-prod`
### Step 2: Create file `cfbackend.tfvars` and include s3 bucket
```
    bucket = "<Bucket of stage run terraform>"
```
### Step 3: Create file `terraform.tfvars` include secret variable
```
    aws-access-key = "<AWS ACCESS KEY>"
    aws-secret-key = "<AWS SECRET KEY>"
    stage-name = "<Name of stage run [staging, prod]>"
    username-database = "<Username of Mysql Database>"
    password-database = "<Password of Mysql Database>"
    backend-image = "<Url image backend on ECR>"
```
### Step 4: Run terraform
- Init: `terraform init -backend-config='cfbackend.tfvars'` to init module, resource of - terraform
- Validate: `terraform validate` to check syntax terraform
- Plan: `terraform plan` to show resource and service will create, change or destroy
- Apply: `terraform apply` to deploy terraform
PART 1: TERRAFORM MODULES
1. What is a Terraform Module?
Interview Definition:

A Terraform module is a collection of Terraform configuration files (.tf) stored in a directory. It helps us organize, reuse, and manage infrastructure code efficiently.

In simple words:

A module is a reusable block of Terraform code. We write the code once and use it multiple times with different values.
Real-world example
Imagine you are constructing 10 houses. Every house requires a bedroom, kitchen, bathroom, and electricity.

Instead of creating a new design for every house, you create one standard design and reuse it.

Similarly, in Terraform, we create one EC2 module and reuse it to launch multiple EC2 instances.
2. Why do we use Terraform Modules?
1. Code Reusability: Write code once and reuse it.
2. Less Code Duplication: Avoid writing the same configuration repeatedly.
3. Easy Maintenance: Update reusable code in one place.
4. Code Organization: Keep Terraform configurations clean.
5. Consistency: Create infrastructure using standardized configurations.
3. Types of Terraform Modules
There are two main types:

A. Root Module
- The main Terraform working directory.
- It contains the main configuration files.
- Terraform commands are executed from this directory.
B. Child Module
- A module called by another module.
- Contains reusable Terraform configurations.
- Can be called multiple times by the root module.
Root Module

Main Terraform directory

Child Module

EC2


Child Module

Security Group




4. Terraform Module Folder Structure
Terraform-Module/
│
├── provider.tf
├── main.tf
├── outputs.tf
│
└── modules/
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf


File	Purpose
provider.tf	Configures AWS provider
Root main.tf	Calls child module
Module main.tf	Creates EC2 instance
variables.tf	Defines input variables
Module outputs.tf	Returns resource information
Root outputs.tf	Displays module outputs

5. Practical: Create a Reusable EC2 Module
Step 1: Configure AWS provider

File: provider.tf
provider "aws" {
  region = "ap-south-1"
}


Step 2: Create module variables

File: modules/ec2/variables.tf
variable "ami_id" {
  description = "AMI ID for EC2"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "instance_name" {
  description = "Name of EC2 instance"
  type        = string
}


Step 3: Create EC2 resource

File: modules/ec2/main.tf
resource "aws_instance" "myec2" {

  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }
}


Step 4: Create module outputs

File: modules/ec2/outputs.tf
output "instance_id" {
  value = aws_instance.myec2.id
}

output "public_ip" {
  value = aws_instance.myec2.public_ip
}


Step 5: Call the module

File: Root main.tf
module "dev_ec2" {

  source = "./modules/ec2"

  ami_id        = "YOUR_VALID_AMI_ID"
  instance_type = "t2.micro"
  instance_name = "Development-Server"
}


module "prod_ec2" {

  source = "./modules/ec2"

  ami_id        = "YOUR_VALID_AMI_ID"
  instance_type = "t2.small"
  instance_name = "Production-Server"
}


Explanation:
- module → Calls a module.
- dev_ec2 → Name of module block.
- source → Location of child module.
- ami_id → Input variable.
- instance_type → EC2 instance type.
- instance_name → Name of EC2 instance.
Notice that we are calling the same EC2 module twice with different values.

Step 6: Access module outputs

File: Root outputs.tf
output "dev_instance_id" {
  value = module.dev_ec2.instance_id
}

output "dev_public_ip" {
  value = module.dev_ec2.public_ip
}

output "prod_instance_id" {
  value = module.prod_ec2.instance_id
}

output "prod_public_ip" {
  value = module.prod_ec2.public_ip
}


6. Terraform Module Execution Commands
terraform init

terraform validate

terraform plan

terraform apply

terraform output

terraform destroy

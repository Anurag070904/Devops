# Terraform Modules and Dependencies

> Notes for DevOps learning and interview preparation\
> Topics: Terraform Modules, Implicit Dependencies, Explicit
> Dependencies

------------------------------------------------------------------------

# Part 1: Terraform Modules

## 1. What is a Terraform Module?

**Interview definition:** A Terraform module is a collection of
Terraform configuration files (`.tf`) stored in a directory. It helps us
organize, reuse, and manage infrastructure code efficiently.

**In simple words:** A module is a reusable block of Terraform code. We
write the code once and use it multiple times with different values.

**Real-world example:** Imagine constructing 10 houses. Instead of
designing every house from scratch, you create one standard design and
reuse it. Similarly, you can create one EC2 module and reuse it to
launch multiple EC2 instances.

## 2. Why do we use Terraform Modules?

1.  **Code reusability:** Write code once and reuse it.
2.  **Less code duplication:** Avoid writing the same configuration
    repeatedly.
3.  **Easy maintenance:** Update reusable code in one place.
4.  **Code organization:** Keep Terraform configurations clean.
5.  **Consistency:** Create infrastructure using standardized
    configurations.

## 3. Types of Terraform Modules

There are two commonly described module roles:

### A. Root Module

-   The main Terraform working directory.
-   Terraform commands are normally executed from this directory.
-   It can call child modules.

### B. Child Module

-   A module called by another module.
-   Contains reusable Terraform configurations.
-   Can be called multiple times by the root module.

## 4. Module folder structure

``` text
Terraform-Module/
├── provider.tf
├── main.tf
├── outputs.tf
└── modules/
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

  File                    Purpose
  ----------------------- ----------------------------------------------
  `provider.tf`           Configures the AWS provider
  Root `main.tf`          Calls the child module
  Module `main.tf`        Defines the EC2 resource
  Module `variables.tf`   Defines input variables
  Module `outputs.tf`     Exposes information about the resource
  Root `outputs.tf`       Displays values returned by the child module

## 5. Practical: Create a reusable EC2 module

This example creates two EC2 instances by calling the same child module
twice. Replace the example AMI ID with a valid AMI ID for your AWS
region.

### Step 1: Configure the AWS provider

File: `provider.tf`

``` hcl
provider "aws" {
  region = "ap-south-1"
}
```

Configure AWS credentials securely, for example through the AWS CLI or
an approved credentials profile. Do not hardcode access keys in
Terraform files.

### Step 2: Define module input variables

File: `modules/ec2/variables.tf`

``` hcl
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
```

Variables let the calling module pass different values to the same
reusable code.

### Step 3: Create the EC2 resource

File: `modules/ec2/main.tf`

``` hcl
resource "aws_instance" "myec2" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }
}
```

-   `var.ami_id` reads the AMI input variable.
-   `var.instance_type` reads the instance type input variable.
-   `var.instance_name` reads the instance name input variable.

### Step 4: Define child-module outputs

File: `modules/ec2/outputs.tf`

``` hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.myec2.id
}

output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.myec2.public_ip
}
```

**Output** exposes useful information from a module, such as an instance
ID or public IP address.

### Step 5: Call the same module twice

File: root `main.tf`

``` hcl
module "dev_ec2" {
  source = "./modules/ec2"

  ami_id        = "ami-REPLACE_WITH_VALID_ID"
  instance_type = "t2.micro"
  instance_name = "Development-Server"
}

module "prod_ec2" {
  source = "./modules/ec2"

  ami_id        = "ami-REPLACE_WITH_VALID_ID"
  instance_type = "t2.small"
  instance_name = "Production-Server"
}
```

Explanation:

-   `module` calls a module.
-   `dev_ec2` and `prod_ec2` are module block names.
-   `source` tells Terraform where the child module is located.
-   The input values can differ for each module call.

The same EC2 module is reused for Development and Production with
different input values.

### Step 6: Read child-module outputs from the root module

File: root `outputs.tf`

``` hcl
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
```

The syntax `module.dev_ec2.instance_id` means: access the `instance_id`
output exposed by the module block named `dev_ec2`.

## 6. Terraform commands

Run these commands from the root module directory:

``` bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
terraform output
```

After practice, remove the resources when they are no longer needed:

``` bash
terraform destroy
```

**Cost and safety note:** Applying this example creates two billable EC2
instances. Use a valid AMI and suitable instance types, review
`terraform plan`, and destroy the resources after practice. Never commit
credentials, private keys, or Terraform state files to Git.

------------------------------------------------------------------------

# Part 2: Terraform Dependencies

## 1. What is a dependency in Terraform?

**Interview definition:** A dependency in Terraform defines the
relationship and ordering between resources, so Terraform knows which
resources must be created, updated, or destroyed before others.

**In simple words:** If one resource needs another resource to exist
first, Terraform must understand that relationship.

**Real-world example:** When constructing a house, the foundation must
be built before the walls, and the walls before the roof. The work has
an order because later steps depend on earlier steps.

## 2. Types of dependencies

There are two main types:

1.  Implicit dependency
2.  Explicit dependency

## 3. Implicit Dependency

**Interview definition:** An implicit dependency is automatically
detected by Terraform when one resource references an attribute of
another resource.

**In simple words:** Terraform sees the reference in the code and
understands the dependency automatically.

### Example: Security Group and EC2

``` hcl
resource "aws_security_group" "web_sg" {
  name = "web-security-group"

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami           = "ami-REPLACE_WITH_VALID_ID"
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    aws_security_group.web_sg.id
  ]
}
```

This reference creates the dependency:

``` hcl
aws_security_group.web_sg.id
```

Terraform understands that the Security Group must be created before the
EC2 instance can use its ID.

**Creation order:** Security Group → EC2 instance.

## 4. Explicit Dependency

**Interview definition:** An explicit dependency is manually declared by
the user using the `depends_on` meta-argument when Terraform cannot
infer the required dependency from resource references.

**In simple words:** We directly tell Terraform which resource must be
created first.

### Example: EC2 depends on an S3 bucket

``` hcl
resource "aws_s3_bucket" "app_data" {
  bucket = "replace-with-a-globally-unique-bucket-name"
}

resource "aws_instance" "app_server" {
  ami           = "ami-REPLACE_WITH_VALID_ID"
  instance_type = "t2.micro"

  depends_on = [
    aws_s3_bucket.app_data
  ]
}
```

Here:

``` hcl
depends_on = [
  aws_s3_bucket.app_data
]
```

We are explicitly telling Terraform to create the S3 bucket before the
EC2 instance, even though the EC2 configuration does not reference a
bucket attribute.

This is an illustrative ordering example. In a real project, use
`depends_on` only when there is a genuine dependency that Terraform
cannot infer.

## 5. Difference between Implicit and Explicit Dependencies

  -----------------------------------------------------------------------
  Implicit dependency                 Explicit dependency
  ----------------------------------- -----------------------------------
  Detected automatically by Terraform Declared manually by the user

  Created through resource references Created using `depends_on`

  Terraform infers the relationship   User specifies the relationship
  from the expression                 

  No `depends_on` is needed for that  `depends_on` is required
  relationship                        
  -----------------------------------------------------------------------

## 6. Important points about dependencies

-   Terraform builds a dependency graph to determine a safe execution
    order.
-   Independent resources can be created or updated in parallel.
-   Terraform generally destroys dependent resources before the
    resources they depend on.
-   Use `depends_on` only when necessary; unnecessary dependencies can
    reduce parallelism.
-   `depends_on` controls Terraform operation ordering. It does not
    guarantee that an application or service is fully ready to accept
    requests.

------------------------------------------------------------------------

# Part 3: Interview Questions and Answers

### Q1. What is a Terraform module?

A Terraform module is a collection of Terraform configuration files used
to organize and reuse infrastructure code.

### Q2. What is the difference between a root module and a child module?

The root module is the main working directory where Terraform commands
are executed. A child module is called by another module to provide
reusable configuration.

### Q3. Can we call the same module multiple times?

Yes. We can call the same module multiple times with different input
values to create different infrastructure resources.

### Q4. What is the purpose of the `source` argument?

The `source` argument specifies the location of the module that
Terraform needs to use.

### Q5. What is a dependency in Terraform?

A dependency defines the relationship and execution order between
resources.

### Q6. What are the two main types of dependencies?

Implicit dependency and explicit dependency.

### Q7. What is an implicit dependency?

It is automatically detected by Terraform when one resource references
an attribute of another resource.

### Q8. What is an explicit dependency?

It is manually declared using the `depends_on` meta-argument.

### Q9. Which argument is used to define an explicit dependency?

The `depends_on` argument.

### Q10. Can Terraform create independent resources in parallel?

Yes. Terraform can create or update independent resources in parallel,
subject to its parallelism settings and provider behavior.

------------------------------------------------------------------------

# Quick Revision

  Concept               Remember this
  --------------------- ------------------------------------------
  Module                Reusable Terraform code
  Root module           Main working directory
  Child module          Reusable module called by another module
  `source`              Location of the module
  Variables             Inputs to a module
  Outputs               Values exposed by a module
  Implicit dependency   Terraform automatically detects it
  Explicit dependency   User defines it using `depends_on`
  Dependency graph      Terraform's resource ordering model

**Remember these interview lines:**

-   **Module:** Write once, reuse multiple times.
-   **Implicit dependency:** Terraform automatically understands the
    dependency.
-   **Explicit dependency:** We manually define the dependency using
    `depends_on`.

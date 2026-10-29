# Terraform Variables and EC2 Demo

## 1. Topic Overview

In this practical, we use Terraform variables to create an AWS EC2
instance. We keep the AWS provider and EC2 resource in `ec2.tf`, and
define reusable input variables in `variables.tf`.

**Security note:** This document intentionally hides real AMI, security
group, and other account-specific identifiers. Replace every placeholder
with a valid value from your own AWS account and region. Never publish
credentials, private keys, or Terraform state files.

## 2. Folder Structure

``` text
Teraform/
├── ec2.tf
└── variables.tf
```

Terraform automatically loads all `.tf` files in the same working
directory. You do not need to import `variables.tf` into `ec2.tf`.

## 3. `ec2.tf` --- Provider and EC2 Resource

``` hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "anurag_instance" {
  ami                    = var.image_id
  instance_type          = var.instance_type
  key_name               = var.key_name
  vpc_security_group_ids = var.vpc_sg

  tags = {
    Name = "anurag_instance"
  }
}
```

### Explanation

-   `provider "aws"` configures Terraform to work with AWS.
-   `region = "ap-south-1"` selects the AWS Mumbai region.
-   `resource "aws_instance" "anurag_instance"` defines an EC2 instance
    Terraform should manage.
-   `ami = var.image_id` uses the AMI ID supplied by the `image_id`
    variable.
-   `instance_type = var.instance_type` uses the instance type supplied
    by a variable.
-   `key_name = var.key_name` references an existing EC2 key pair in the
    selected region.
-   `vpc_security_group_ids = var.vpc_sg` supplies a list of security
    group IDs.
-   `tags` adds a Name tag to the instance.

**Important:** `var.image_id` and other variable references must not be
enclosed in quotation marks. For example, use `ami = var.image_id`, not
`ami = "var.image_id"`.

## 4. `variables.tf` --- Declare Variables

Use placeholders instead of publishing real resource identifiers:

``` hcl
variable "image_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
  default     = "ami-REPLACE_WITH_VALID_AMI_ID"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "key_name" {
  description = "Name of an existing EC2 key pair"
  type        = string
  default     = "REPLACE_WITH_KEY_PAIR_NAME"
}

variable "vpc_sg" {
  description = "Security group IDs for the EC2 instance"
  type        = list(string)
  default     = ["sg-REPLACE_WITH_SECURITY_GROUP_ID"]
}
```

Before running Terraform, replace each placeholder with the correct
value in your own AWS account. The AMI, key pair, and security group
must be compatible with the selected region and VPC.

### Why use variables?

Variables help keep configuration reusable and easier to update. For
example, you can change the instance type in one place rather than
editing the resource block itself.

## 5. Important Terraform Variable Rules

### Correct variable references

Correct:

``` hcl
ami                    = var.image_id
instance_type          = var.instance_type
vpc_security_group_ids = var.vpc_sg
```

Incorrect:

``` hcl
ami                    = "var.image_id"
instance_type          = "var.instance_type"
vpc_security_group_ids = [var.vpc_sg]
```

The quoted examples pass literal text rather than evaluating the
Terraform variable. The last example incorrectly wraps a list variable
inside another list.

### List type

The EC2 argument `vpc_security_group_ids` expects a list of strings.
Therefore, declare the variable as:

``` hcl
variable "vpc_sg" {
  type    = list(string)
  default = ["sg-REPLACE_WITH_SECURITY_GROUP_ID"]
}
```

Then reference it directly:

``` hcl
vpc_security_group_ids = var.vpc_sg
```

Do not add square brackets around `var.vpc_sg`, because it is already a
list.

### AMI ID

An AMI ID must be a single valid AMI identifier. Do not paste the same
AMI ID twice or include extra characters. The actual AMI must exist in
the selected region.

## 6. Commands Used in the Practical

Run these commands from the directory containing `ec2.tf` and
`variables.tf`:

``` bash
terraform --version
ls -la
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
```

### What each command does

-   `terraform --version` --- displays the installed Terraform version.
-   `ls -la` --- lists files, including hidden files.
-   `terraform fmt` --- formats Terraform configuration files.
-   `terraform init` --- initializes the directory and downloads
    required providers.
-   `terraform validate` --- checks configuration syntax and
    consistency.
-   `terraform plan` --- previews the changes Terraform proposes.
-   `terraform apply` --- applies changes after you review and confirm
    the plan.

`sudo apt update` is an operating-system package-list update command; it
is not a required Terraform workflow command.

## 7. IAM Permissions

The EC2 machine running Terraform needs permission to perform the
required AWS operations. A common practice is to attach an IAM role to
that machine and grant least-privilege permissions.

Avoid using full administrator access for a simple demo unless it is
specifically required and approved. Do not hardcode long-lived AWS
access keys in Terraform files.

## 8. Troubleshooting the Errors from the Demo

### Error A: Invalid AMI ID

Example:

``` text
InvalidAMIID.Malformed: Invalid id: "var.image_id"
```

**Possible cause:** The variable was written as a quoted string:

``` hcl
ami = "var.image_id"
```

**Fix:**

``` hcl
ami = var.image_id
```

Also check that the variable value is one valid AMI ID for the selected
region, not a duplicated or placeholder value.

### Error B: Invalid instance type

Example:

``` text
Invalid value 'var.instance_type' for InstanceType
```

**Possible cause:** Terraform passed `"var.instance_type"` as literal
text.

**Fix:**

``` hcl
instance_type = var.instance_type
```

Check that the variable has a valid value, such as `t3.micro`.

### Error C: Incorrect attribute value type for security groups

Example:

``` text
Inappropriate value for attribute "vpc_security_group_ids":
element 0: string required, but have tuple
```

**Possible cause:** A list variable was wrapped in another list:

``` hcl
vpc_security_group_ids = [var.vpc_sg]
```

**Fix:**

``` hcl
vpc_security_group_ids = var.vpc_sg
```

The variable should be declared as `list(string)`.

### Error D: Key pair or security group not found

**Possible cause:** The key pair or security group does not exist in the
selected region, or it is incompatible with the selected VPC/subnet.

**Fix:** Verify the key pair and security group in the AWS console in
`ap-south-1`, then update the variable values.

## 9. Verify the Result

After a successful apply:

``` bash
terraform show
terraform state list
```

The state list should show a resource address similar to:

``` text
aws_instance.anurag_instance
```

You can also check the EC2 console in the AWS Mumbai region. Do not
share raw Terraform state publicly; it can contain sensitive
information.

## 10. Clean Up the Demo

If the EC2 instance was created only for practice and is no longer
needed, first review the proposed destruction:

``` bash
terraform plan -destroy
```

If you are certain the instance can be removed, run:

``` bash
terraform destroy
```

Review the changes and confirm only when appropriate. Destroying
resources can cause data loss or service interruption.

## 11. Best Practices

1.  Use variables instead of hardcoding values throughout resource
    blocks.
2.  Specify variable types such as `string` and `list(string)` to
    improve clarity and validation.
3.  Do not put quotes around Terraform variable references such as
    `var.image_id`.
4.  Do not wrap a list variable in another list unnecessarily.
5.  Check that AMI IDs, key pairs, and security groups belong to the
    intended AWS region and VPC.
6.  Run `terraform fmt`, `terraform validate`, and `terraform plan`
    before applying.
7.  Use least-privilege IAM permissions.
8.  Keep credentials, private keys, state files, and backups out of
    public Git repositories.
9.  Replace all placeholders before running the configuration.
10. Check your AWS resources and costs after the demo.

Suggested `.gitignore` entries:

``` gitignore
*.tfstate
*.tfstate.*
.terraform/
```

Keep `.terraform.lock.hcl` under version control in most projects so
provider selections are reproducible.

## 12. Interview Revision

**Q: Why do we use Terraform variables?**\
A: Variables make configurations reusable and easier to update without
changing resource blocks.

**Q: How do you reference a variable?**\
A: Use `var.variable_name`, for example `var.instance_type`, without
quotation marks.

**Q: Why did the security group list produce a type error?**\
A: The variable was already a list, but it was wrapped in another list.
Referencing `var.vpc_sg` directly fixes the type mismatch.

**Q: What does `terraform plan` do?**\
A: It previews proposed infrastructure changes before they are applied.

**Q: What does `terraform apply` do?**\
A: It applies the planned changes to the infrastructure after
confirmation when required.

**Q: Why should state files not be committed to public GitHub?**\
A: State files can contain sensitive resource attributes or secrets, so
they must be protected.

------------------------------------------------------------------------

## Summary

This practical demonstrates how to: - Configure the AWS provider in
`ec2.tf`. - Define variables separately in `variables.tf`. - Reference
variables correctly. - Pass security group IDs as a list of strings. -
Initialize, validate, plan, and apply Terraform configuration. -
Troubleshoot common AMI, instance type, and list-type errors. - Keep
sensitive identifiers and state files out of public repositories.

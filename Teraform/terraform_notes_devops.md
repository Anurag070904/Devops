# Terraform Notes --- DevOps

## 1. What Is Terraform?

**Terraform is an Infrastructure as Code (IaC) tool** used to define,
create, update, and manage infrastructure through configuration files
instead of creating every resource manually in a cloud console.

Terraform is developed by HashiCorp and works with many platforms
through providers, including AWS, Azure, Google Cloud, Kubernetes, and
other services.

### Interview definition

> Terraform is an Infrastructure as Code tool used to provision and
> manage infrastructure through configuration files. It helps automate
> the creation and management of resources such as virtual machines,
> networks, databases, and Kubernetes clusters.

### Simple example

Suppose a company needs: - 3 EC2 instances - 1 VPC - 2 security groups -
1 load balancer - 1 database

We could create these resources manually in AWS. With Terraform, we
describe the desired infrastructure in `.tf` files and use Terraform
commands to plan and apply the changes.

**Real-world example (Marathi/Hindi):** Ekda infrastructure cha code
lihila ki, to reuse karun development, testing ani production
environments madhye similar infrastructure create karta yeto.
Environment-specific values, permissions, and configurations still need
to be handled correctly.

------------------------------------------------------------------------

## 2. Why Do We Use Terraform?

We use Terraform to: 1. **Automate infrastructure creation** and reduce
repetitive manual work. 2. **Create consistent environments** across
development, testing, and production. 3. **Manage infrastructure as
code** so configuration can be reviewed and version-controlled. 4.
**Preview changes** with `terraform plan` before applying them. 5.
**Reuse configuration** through variables and modules. 6. **Manage
different platforms** through providers. 7. **Track resources** using
Terraform state.

------------------------------------------------------------------------

## 3. Advantages of Terraform

-   **Automation:** Creates, updates, and manages supported
    infrastructure resources.
-   **Consistency:** Helps create environments with repeatable
    configurations.
-   **Version control:** Terraform files can be stored in GitHub or
    GitLab, allowing teams to review changes and see history.
-   **Plan before apply:** The `terraform plan` command previews
    proposed changes.
-   **Reusable modules:** Common infrastructure configurations can be
    reused across projects.
-   **Multi-platform support:** Providers allow Terraform to manage many
    cloud and service platforms.
-   **Dependency handling:** Terraform builds a dependency graph and
    generally creates or updates resources in a suitable order.
-   **State tracking:** Terraform records mappings between configuration
    resources and real infrastructure.

### Important limitation

Terraform does not remove the need for security reviews, backups,
monitoring, or testing. A wrong configuration or an unsafe apply can
still cause downtime, unwanted changes, or data loss.

------------------------------------------------------------------------

## 4. Terraform vs. Docker vs. Kubernetes

  Tool         Main purpose
  ------------ -----------------------------------------------------
  Terraform    Creates and manages infrastructure
  Docker       Packages and runs applications in containers
  Kubernetes   Orchestrates and manages containerized applications

**Example:** Terraform can create an AWS EKS cluster and supporting
infrastructure. Kubernetes then manages the applications running inside
that cluster. Docker can be used to build container images for those
applications.

------------------------------------------------------------------------

## 5. Terraform Workflow / Lifecycle

The common Terraform workflow is:

1.  **Write configuration** --- create `.tf` files describing the
    desired infrastructure.
2.  **`terraform init`** --- initializes the working directory and
    installs required providers and modules.
3.  **`terraform fmt`** --- formats Terraform configuration files.
4.  **`terraform validate`** --- checks configuration syntax and
    internal consistency.
5.  **`terraform plan`** --- previews proposed changes.
6.  **`terraform apply`** --- applies the changes after confirmation,
    creating or updating resources.
7.  **Update configuration as needed** --- plan and apply changes when
    infrastructure requirements change.
8.  **`terraform destroy` (optional)** --- destroys resources managed by
    the configuration when that is intentionally required.

### Important commands

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `terraform --version`               Displays the installed Terraform
                                      version

  `terraform init`                    Initializes the working directory

  `terraform fmt`                     Formats `.tf` files

  `terraform validate`                Validates configuration syntax and
                                      consistency

  `terraform plan`                    Previews changes

  `terraform apply`                   Applies planned changes

  `terraform show`                    Displays the current state or a
                                      saved plan

  `terraform output`                  Displays output values

  `terraform destroy`                 Plans and applies destruction of
                                      managed resources
  -----------------------------------------------------------------------

**Note:** `terraform destroy` is optional. Do not run it casually,
because it can delete infrastructure and potentially cause data loss.

------------------------------------------------------------------------

## 6. Blocks in Terraform

A **block** is a structural section in a Terraform configuration.
Different block types serve different purposes.

### 6.1 Terraform block

Defines Terraform requirements and configuration, such as the required
Terraform version and providers.

``` hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

The version constraints are examples. Select versions that are
compatible with your project and update them deliberately.

### 6.2 Provider block

Configures the provider Terraform uses. For AWS, it can specify the
region.

``` hcl
provider "aws" {
  region = "ap-south-1"
}
```

`ap-south-1` is the AWS Mumbai region.

### 6.3 Resource block

Defines an infrastructure resource Terraform should create or manage.

``` hcl
resource "aws_instance" "demo" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"
}
```

-   `resource` --- block type.
-   `aws_instance` --- resource type.
-   `demo` --- local name used to refer to this resource.
-   `ami` and `instance_type` --- arguments inside the block.

Replace the example AMI ID with a valid AMI ID for the target region and
intended operating system.

### 6.4 Variable block

Defines an input value that can be reused.

``` hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}
```

Use it in a resource:

``` hcl
resource "aws_instance" "demo" {
  ami           = "ami-xxxxxxxx"
  instance_type = var.instance_type
}
```

### 6.5 Output block

Displays useful information after Terraform applies the configuration.

``` hcl
output "server_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.demo.id
}
```

### 6.6 Data block

Reads information about existing resources or external data; it does not
create the resource represented by the data source.

``` hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

The nested `filter` block helps search for a matching AMI. The exact
filter should be checked for the required architecture and operating
system.

### 6.7 Module block

Calls a reusable collection of Terraform configuration.

``` hcl
module "network" {
  source = "./modules/network"
}
```

The local directory must exist and contain a valid module configuration.

### Blocks vs. arguments

``` hcl
resource "aws_instance" "demo" {
  instance_type = "t3.micro"
}
```

-   **Block:** `resource "aws_instance" "demo" { ... }`
-   **Argument:** `instance_type = "t3.micro"`

The block groups configuration; arguments specify values inside it.

------------------------------------------------------------------------

## 7. Terraform `lifecycle` Block

The `lifecycle` block is different from the overall Terraform workflow.
It controls specific behavior for an individual resource.

### Common lifecycle arguments

-   **`create_before_destroy`** --- asks Terraform to create a
    replacement before destroying the old resource when replacement is
    required and the resource/provider supports this ordering.
-   **`prevent_destroy`** --- rejects a plan that would destroy a
    resource while this rule remains in the configuration. It is not a
    complete protection against all deletion scenarios.
-   **`ignore_changes`** --- tells Terraform to ignore changes to
    specified resource attributes during update planning.
-   **`replace_triggered_by`** --- triggers replacement when specified
    managed resources or attributes change.

Example:

``` hcl
resource "aws_instance" "demo" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  lifecycle {
    prevent_destroy = true
  }
}
```

This can help prevent accidental planned destruction while the lifecycle
rule is present. Use lifecycle settings carefully and understand their
consequences before using them in production.

------------------------------------------------------------------------

## 8. What Is `terraform.tfstate`?

**Interview definition:**

> Terraform state is a record that maps Terraform configuration
> resources to real infrastructure and stores information Terraform
> needs to manage those resources.

After creating an EC2 instance, Terraform state can record the resource
address, AWS instance ID, and other resource attributes.

For example, the resource address might be:

``` text
aws_instance.demo
```

The actual EC2 instance ID would look something like
`i-0123456789abcdef0`; this is only an example, not a real ID from your
environment.

### Why is state important?

-   It helps Terraform associate a configuration resource with the real
    resource it manages.
-   It records attributes Terraform needs for future operations.
-   Terraform uses state when planning changes.
-   It helps Terraform understand what it already manages.

### Inspect state

``` bash
terraform show
```

To view raw local state (be careful because it may contain sensitive
information):

``` bash
cat terraform.tfstate
```

### Important precautions

-   Do not casually delete `terraform.tfstate`.
-   Do not manually edit the state file unless you understand the
    state-management workflow.
-   Do not commit state files to a public repository.
-   Protect state files because they may contain sensitive values.
-   In team environments, consider a properly configured remote backend,
    such as an S3 backend, with access controls, encryption, versioning,
    and state locking where supported and configured.

------------------------------------------------------------------------

## 9. What Is `terraform.tfstate.backup`?

With the **local backend**, Terraform typically saves the previous state
snapshot in `terraform.tfstate.backup` when it writes a new state file.
The exact backup behavior depends on the backend and operation.

### Difference

  -----------------------------------------------------------------------
  `terraform.tfstate`                 `terraform.tfstate.backup`
  ----------------------------------- -----------------------------------
  Current local state snapshot        Previous state snapshot, when a
                                      backup is created

  Used for normal Terraform state     May help recover or inspect an
  tracking                            earlier snapshot

  Updated as Terraform writes state   May be replaced by a later backup
  -----------------------------------------------------------------------

A backup is not necessarily created on the very first apply. Do not
assume it always exists.

### What if the files are deleted?

-   Deleting `terraform.tfstate` does **not** automatically delete the
    EC2 instance from AWS, but Terraform loses its normal local mapping
    to that resource.
-   Without state, Terraform may propose creating a duplicate or may not
    manage the existing resource correctly until state is recovered or
    reconciled.
-   Deleting only `terraform.tfstate.backup` removes that backup copy;
    the current state can continue working if it is intact.
-   Deleting both removes those local state snapshots and makes recovery
    harder.

### Git ignore

For a local Terraform project, a `.gitignore` file can include:

``` gitignore
*.tfstate
*.tfstate.*
.terraform/
```

Keep `.terraform.lock.hcl` under version control in most projects; it
records selected provider dependency versions and checksums. Do not
blindly ignore all Terraform-related files.

------------------------------------------------------------------------

## 10. Practical Demo: Create an EC2 Instance Using Terraform

This demo follows the practical exercise: launch one EC2 instance to run
Terraform, then use Terraform from that machine to create a second EC2
instance.

### Architecture

``` text
Your terminal
     |
     v
EC2 #1 — Terraform control machine
     |
     | IAM role permissions + AWS provider
     v
AWS API
     |
     v
EC2 #2 — Instance created by Terraform
```

### Step 1: Launch the control EC2 instance

Launch an EC2 instance to use as the Terraform control machine. The
example uses Ubuntu, so the installation commands below use
Ubuntu/Debian package conventions.

### Step 2: Install Terraform

Install Terraform using HashiCorp's official installation instructions
for your operating system and distribution. Then verify it:

``` bash
terraform --version
```

### Step 3: Configure AWS permissions securely

Attach an IAM role to the control EC2 instance. Grant only the
permissions needed for the resources Terraform must manage, plus any
necessary supporting permissions.

Avoid using full administrator access for a simple demo unless there is
a specific, justified requirement. The EC2 instance can obtain temporary
credentials from its IAM role, so long-lived access keys generally
should not be hardcoded in the `.tf` file.

### Step 4: Create the configuration file

``` bash
mkdir terraform-demo
cd terraform-demo
vim ec2.tf
```

Press `i` in Vim to enter insert mode, type the configuration, then
press `Esc`, type `:wq`, and press Enter to save and exit.

### Step 5: Write the Terraform configuration

``` hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "demo" {
  ami                    = "ami-xxxxxxxx"
  instance_type          = "t3.micro"
  key_name               = "launch_wizard"
  vpc_security_group_ids = ["sg-xxxxxxxx"]

  tags = {
    Name = "demo"
  }
}
```

**Before applying:** - Replace `ami-xxxxxxxx` with a valid AMI ID in
`ap-south-1`. - Replace `sg-xxxxxxxx` with the correct security group
ID. - Make sure the key pair named `launch_wizard` exists in the same
region. - Make sure the chosen AMI, instance type, subnet, security
group, and account are compatible. - Ensure the IAM role has the
necessary permissions to create and manage the instance and related
resources. - The example uses a public AMI ID placeholder; do not copy
it literally.

### Step 6: Check the file

``` bash
ls -la
cat ec2.tf
```

### Step 7: Initialize Terraform

``` bash
terraform init
```

This initializes the working directory and installs the AWS provider.

### Step 8: Update Ubuntu packages only if needed

``` bash
sudo apt update
```

This updates Ubuntu's package lists. It is not a required Terraform
lifecycle step and does not initialize Terraform.

### Step 9: Format and validate

``` bash
terraform fmt
terraform validate
```

### Step 10: Preview changes

``` bash
terraform plan
```

Review the proposed changes before applying them.

### Step 11: Create the EC2 instance

``` bash
terraform apply
```

Review the plan and type `yes` when prompted if you intend to proceed.
Terraform should create the resource if the configuration, permissions,
quotas, and AWS settings are valid.

### Step 12: Verify the result

Check the AWS EC2 console in the Mumbai region, or inspect Terraform's
output:

``` bash
terraform show
terraform state list
```

The state list should include the managed resource address, such as:

``` text
aws_instance.demo
```

If the apply succeeds, the AWS console should show the new instance. The
actual instance ID will be assigned by AWS.

### Step 13: Clean up only when finished

If this was a temporary practice instance and you no longer need it, you
can review the destruction plan and destroy it:

``` bash
terraform plan -destroy
terraform destroy
```

Review the proposed deletions carefully before confirming. Do not run
destroy if you need to keep the instance or its data.

------------------------------------------------------------------------

## 11. Common Mistakes to Avoid

1.  **Wrong capitalization:** Terraform block keywords are
    case-sensitive. Use `provider`, not `Provider`.
2.  **Incorrect braces:** Put `tags` inside the `resource` block.
3.  **Wrong region:** The AMI, key pair, and security group must be
    valid for the intended region.
4.  **Insufficient IAM permissions:** Terraform must have permissions
    for the actions it performs.
5.  **Skipping the plan review:** Always inspect planned changes before
    applying.
6.  **Exposing state files:** State can contain sensitive data; do not
    publish it.
7.  **Deleting state casually:** The real infrastructure may remain in
    AWS even if local state is lost.
8.  **Assuming every update replaces a resource:** Terraform may update
    in place or replace a resource depending on the changed attribute
    and provider behavior.
9.  **Forgetting cleanup:** Temporary cloud resources can continue to
    incur charges.
10. **Using placeholders literally:** Replace example AMI and security
    group IDs before applying.

------------------------------------------------------------------------

## 12. Frequently Asked Terraform Interview Questions

### Q1. What is Terraform?

Terraform is an Infrastructure as Code tool used to provision and manage
infrastructure through configuration files.

### Q2. Why do we use Terraform?

We use Terraform to automate infrastructure provisioning, improve
consistency, reuse configuration, review changes, and manage
infrastructure through version control.

### Q3. What is a provider?

A provider is a plugin that lets Terraform interact with a platform or
service, such as AWS.

### Q4. What is a resource?

A resource is a configuration block that defines an infrastructure
object Terraform should manage, such as an EC2 instance.

### Q5. What does `terraform init` do?

It initializes the Terraform working directory and downloads required
providers and modules.

### Q6. What is the difference between `terraform plan` and `terraform apply`?

`terraform plan` previews the proposed changes. `terraform apply`
executes those changes after confirmation when required.

### Q7. What is Terraform state?

State records the mapping between Terraform configuration resources and
real infrastructure, along with attributes Terraform needs to manage
them.

### Q8. What is `terraform.tfstate.backup`?

With the local backend, it is typically a previous state snapshot
created when Terraform writes updated state. Its availability depends on
the backend and operation.

### Q9. What is the `lifecycle` block?

It configures resource-specific behavior, including creation order,
destruction protection, ignored changes, and replacement triggers.

### Q10. What is a Terraform module?

A module is a reusable collection of Terraform configuration that can be
called from another configuration.

### Q11. What is the difference between Terraform, Docker, and Kubernetes?

Terraform manages infrastructure, Docker packages and runs applications
in containers, and Kubernetes orchestrates containerized applications.

### Q12. Does deleting `terraform.tfstate` delete the EC2 instance?

No. Deleting the state file does not itself delete the AWS instance, but
it removes Terraform's normal local tracking information for that
resource.

------------------------------------------------------------------------

## Final Revision

Remember these key concepts:

-   **Terraform:** Infrastructure as Code.
-   **Provider:** Connects Terraform to a platform such as AWS.
-   **Resource:** Infrastructure Terraform manages.
-   **Variable:** Input to make configuration reusable.
-   **Output:** Displays useful values.
-   **Data source:** Reads existing information.
-   **Module:** Reusable Terraform configuration.
-   **Lifecycle block:** Controls specific resource behavior.
-   **`terraform.tfstate`:** Current state tracking information.
-   **`terraform.tfstate.backup`:** A previous local state snapshot when
    one is created.
-   **`init → fmt → validate → plan → apply`:** A common working
    sequence.
-   **`destroy`:** Removes managed resources when intentionally
    requested.

**Best practice:** Keep Terraform configuration in Git, keep sensitive
state out of public repositories, use least-privilege IAM permissions,
review plans, and verify changes before applying them.

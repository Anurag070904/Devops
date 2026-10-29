# Terraform Notes: Security Groups and Heredoc

## Part 1: Security Groups in Terraform

### 1. What is an AWS Security Group?

An **AWS Security Group is a virtual firewall** that controls inbound
and outbound network traffic for supported resources such as EC2
instances.

-   **Inbound rules:** Control traffic coming into the resource.
-   **Outbound rules:** Control traffic leaving the resource.

  Port   Purpose
  ------ --------------------------------------------
  22     SSH --- remote Linux server administration
  80     HTTP --- website traffic
  443    HTTPS --- secure website traffic

**Security tip:** Avoid exposing SSH port 22 to everyone (`0.0.0.0/0`).
Restrict it to a trusted IP or approved network whenever possible.

### 2. Create a security group

Create `security_group.tf`:

``` hcl
resource "aws_security_group" "demo_sg" {
  name        = "demo-security-group"
  description = "Security group for demo EC2"
  vpc_id      = var.vpc_id

  # Allow SSH only from a trusted IPv4 address/range
  ingress {
    description = "SSH access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.my_ip_cidr]
  }

  # Allow HTTP website traffic
  ingress {
    description = "HTTP access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow outbound IPv4 traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "demo-security-group"
  }
}
```

### 3. Declare variables

Add to `variables.tf`:

``` hcl
variable "vpc_id" {
  description = "VPC ID where the security group will be created"
  type        = string
}

variable "my_ip_cidr" {
  description = "Trusted public IPv4 address in CIDR notation"
  type        = string
}
```

Supply values in `terraform.tfvars`:

``` hcl
vpc_id     = "vpc-REPLACE_WITH_YOUR_VPC_ID"
my_ip_cidr = "REPLACE_WITH_YOUR_PUBLIC_IP/32"
```

Replace these placeholders with valid values from your AWS account.
`/32` represents one IPv4 address. The security group and the EC2
instance using it must belong to the same VPC.

### 4. Attach the security group to EC2

In `ec2.tf`, use the security group's ID:

``` hcl
resource "aws_instance" "web_server" {
  ami           = var.image_id
  instance_type = var.instance_type
  key_name      = var.key_name

  vpc_security_group_ids = [
    aws_security_group.demo_sg.id
  ]

  tags = {
    Name = "web-server"
  }
}
```

`aws_security_group.demo_sg.id` references the ID of the security group
Terraform creates. Terraform understands the dependency and can create
the security group before the EC2 instance needs it.

If you need other security groups too, include their IDs in the list.
Review the plan carefully because some EC2 configuration changes may
require instance replacement.

### 5. Commands

Run these commands from the directory containing your `.tf` files:

``` bash
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
```

-   `terraform fmt`: formats configuration.
-   `terraform init`: initializes the directory and downloads providers.
-   `terraform validate`: checks configuration syntax and consistency.
-   `terraform plan`: previews proposed changes.
-   `terraform apply`: applies changes after you review and confirm the
    plan.

After applying, check the security group and its rules in the AWS EC2
console, and confirm it is attached to the intended instance.

------------------------------------------------------------------------

## Part 2: Heredoc in Terraform

### 1. What is Heredoc?

**Heredoc is Terraform syntax for writing a multi-line string.** It is
useful for scripts, configuration files, and text that would be awkward
to write as one long quoted string.

**Interview definition:** Heredoc in Terraform is a syntax for defining
multi-line strings. It is commonly used to supply shell scripts, EC2
user data, or configuration content without manually escaping every
newline.

### 2. Simple example

``` hcl
output "demo_message" {
  value = <<EOF
Hello!
Welcome to Terraform.
Keep practising DevOps.
EOF
}
```

-   `<<EOF` starts the multi-line string.
-   The lines between the delimiters are the string content.
-   `EOF` on its own line ends the string.
-   The delimiter can be another label, such as `EOT`, if the opening
    and closing labels match.

### 3. `<<EOF` vs. `<<-EOF`

``` hcl
output "demo_message" {
  value = <<-EOT
    Terraform makes infrastructure automation easier.
    Keep learning, bhau!
  EOT
}
```

-   `<<EOF` preserves indentation as written.
-   `<<-EOF` allows Terraform to remove common leading indentation from
    the lines, making the string easier to indent inside an HCL block.

### 4. Real-world example: EC2 user data

Heredoc can supply a multi-line startup script through EC2's `user_data`
argument:

``` hcl
resource "aws_instance" "web_server" {
  ami           = var.image_id
  instance_type = var.instance_type

  user_data = <<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get install -y nginx
    systemctl enable --now nginx
  EOF

  tags = {
    Name = "web-server"
  }
}
```

This script is for a compatible Ubuntu-based AMI. Other operating
systems may require different commands. EC2 passes the user-data script
to the instance; the guest operating system's startup mechanism executes
it.

**Important:** Heredoc creates a string; it does not execute shell
commands by itself. The receiving argument or program determines what
happens to that string.

### 5. Heredoc best practices

-   Use it when multi-line text is easier to read than escaped newline
    characters.
-   Keep the closing delimiter on its own line.
-   Match the opening and closing delimiter labels.
-   Do not put passwords or access keys in user data; user data may be
    accessible to principals with sufficient permissions.
-   Test startup scripts carefully and review plans before applying
    changes.

------------------------------------------------------------------------

## Part 3: Quick Comparison

  -----------------------------------------------------------------------
  Concept                             Purpose
  ----------------------------------- -----------------------------------
  Security group                      Controls network traffic to and
                                      from supported AWS resources

  Heredoc                             Represents multi-line text inside
                                      Terraform

  `ingress`                           Defines incoming security-group
                                      rules

  `egress`                            Defines outgoing security-group
                                      rules

  `user_data`                         Can deliver a startup script to an
                                      EC2 instance

  `terraform plan`                    Previews proposed infrastructure
                                      changes

  `terraform apply`                   Applies reviewed changes
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## Part 4: Interview Questions

**Q1. What is a security group?**\
An AWS Security Group is a virtual firewall that controls inbound and
outbound traffic for resources such as EC2 instances.

**Q2. What is the difference between ingress and egress?**\
Ingress rules control incoming traffic; egress rules control outgoing
traffic.

**Q3. What is CIDR notation?**\
CIDR notation represents an IP address range. For example,
`192.0.2.10/32` represents one IPv4 address. This is an example address,
not a real user IP.

**Q4. How do you attach a Terraform-created security group to EC2?**\
Reference its ID in `vpc_security_group_ids`, for example
`aws_security_group.demo_sg.id`.

**Q5. What is Heredoc?**\
Heredoc is a syntax for writing multi-line strings in Terraform.

**Q6. What is the difference between `<<EOF` and `<<-EOF`?**\
`<<EOF` preserves indentation as written; `<<-EOF` allows common leading
indentation to be removed.

**Q7. Does Heredoc execute a shell script?**\
No. Heredoc creates a string. If the string is supplied to EC2
`user_data`, the instance's startup mechanism may execute it.

------------------------------------------------------------------------

## Final Revision

-   **Security group:** Virtual firewall for network traffic.
-   **Ingress:** Incoming traffic.
-   **Egress:** Outgoing traffic.
-   **Port 22:** SSH; restrict it to trusted sources.
-   **Port 80:** HTTP.
-   **Port 443:** HTTPS.
-   **Heredoc:** Multi-line string syntax in Terraform.
-   **`<<-EOF`:** Heredoc form that permits common indentation removal.
-   **Security best practice:** Use least privilege, keep secrets out of
    public notes, and review `terraform plan` before `terraform apply`.

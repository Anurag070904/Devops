# Terraform Remote State and Backend

## 1. What is Terraform State?

**Interview Definition:**

Terraform State is a file that stores information about the infrastructure managed by Terraform. It helps Terraform track resources and their actual IDs and attributes.

**In simple words:**

Terraform State is like a record book that remembers which resources Terraform has created and manages.

Default filename:

```text
terraform.tfstate
```

Example: If we create 2 EC2 instances and 1 S3 bucket, Terraform records information about these resources in the state file.

## 2. Why do we need Remote State?

By default, Terraform stores its state file locally on our machine.

Example:

```text
Terraform/
├── main.tf
├── provider.tf
└── terraform.tfstate
```

### Problem with Local State

Imagine three DevOps engineers are working on the same AWS infrastructure.

- Engineer 1 works on their laptop.
- Engineer 2 works on their laptop.
- Engineer 3 works on their laptop.

If each engineer maintains a separate local state file, their state information can become different or outdated.

This can cause infrastructure management problems.

### Solution: Remote State

Instead of storing the state file only on an individual machine, we store it in a common remote location such as AWS S3.

Now all authorized engineers can access the same remote state.

**Real-world example:**

Imagine three employees maintaining three different registers for the same company.

There can be differences between those registers.

Instead, the company maintains one common register that authorized employees can access.

This is similar to Terraform Remote State.

## 3. Why do we store Terraform State remotely?

There are five major reasons:

1. **Centralized Storage:** All team members can access the same state.
2. **Team Collaboration:** Multiple engineers can work with shared infrastructure.
3. **State Locking:** Helps prevent conflicting simultaneous Terraform operations.
4. **Backup and Recovery:** Remote storage features such as S3 versioning help recover previous state versions.
5. **Security:** We can control who can access the state using IAM permissions.

**Important:** Remote storage does not automatically guarantee locking, versioning or encryption. These features must be configured appropriately.

## 4. What is Terraform Backend?

**Interview Definition:**

A Terraform Backend defines where and how Terraform stores and manages its state file.

**In simple words:**

Backend tells Terraform where to save the `terraform.tfstate` file.

By default, Terraform uses the Local Backend.

If we want to store the state file in AWS S3, we configure an S3 Backend.

## 5. Types of Terraform Backend

Some commonly used backends are:

| Backend | Storage |
|---|---|
| Local | Local machine |
| S3 | AWS S3 |
| azurerm | Azure Blob Storage |
| gcs | Google Cloud Storage |
| Remote | Terraform Cloud |

## 6. Provider vs Backend

| Provider | Backend |
|---|---|
| Creates and manages infrastructure | Stores and manages Terraform state |
| Communicates with cloud services | Manages state storage |
| Example: AWS provider | Example: S3 backend |
| Uses `provider` block | Uses `terraform` block |

Example of AWS Provider:

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

Example of S3 Backend:

```hcl
terraform {
  backend "s3" {
    bucket       = "my-terraform-state"
    key          = "dev/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile = true
  }
}
```

## 7. Practical: Configure Terraform Backend using AWS S3

Our goal is to store the Terraform state file in an AWS S3 bucket instead of local storage.

### Step 1: Create an S3 Bucket

Create an S3 bucket in AWS.

Example:

```text
Bucket Name: my-terraform-state-demo
Region: ap-south-1
```

Important configurations:

- Enable Bucket Versioning.
- Enable encryption.
- Block public access.
- Configure appropriate IAM permissions.

### Step 2: Configure Terraform Backend

Create or update the root-level `backend.tf` file.

```hcl
terraform {
  backend "s3" {
    bucket       = "my-terraform-state-demo"
    key          = "dev/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile = true
  }
}
```

Explanation:

| Argument | Meaning |
|---|---|
| `backend "s3"` | Specifies AWS S3 as backend |
| `bucket` | S3 bucket name |
| `key` | Path where state file is stored |
| `region` | AWS region |
| `use_lockfile` | Enables S3 lockfile-based state locking |

For example:

```hcl
key = "dev/terraform.tfstate"
```

This means Terraform stores the state at:

```text
s3://my-terraform-state-demo/dev/terraform.tfstate
```

### Step 3: Initialize Terraform

Run:

```bash
terraform init
```

Terraform initializes the S3 backend.

If you already have local state, Terraform may ask whether you want to migrate the existing state to S3.

Choose migration if you intend to move your existing state.

### Step 4: Verify Remote State

After applying your Terraform configuration, open your S3 bucket.

You should find:

```text
my-terraform-state-demo/
│
└── dev/
    └── terraform.tfstate
```

Your active Terraform state is now stored remotely.

## 8. What is State Locking?

**Interview Definition:**

State Locking prevents multiple Terraform operations from modifying the same state simultaneously.

Example:

1. Engineer 1 runs `terraform apply`.
2. Terraform acquires the state lock.
3. Engineer 2 tries to run `terraform apply`.
4. Engineer 2 cannot acquire the lock while Engineer 1 holds it.
5. After Engineer 1 finishes, the lock is released.

Example configuration:

```hcl
use_lockfile = true
```

This enables S3 lockfile-based locking.

**Note:** S3 native lockfile support requires Terraform 1.10 or later.

State locking helps prevent conflicting Terraform operations. It does not prevent manual changes through the AWS Console.

## 9. Do we need a separate Backend Block for every resource?

No!

We need only one backend configuration per Terraform root module.

Example:

```text
Terraform/
│
├── backend.tf
├── provider.tf
├── ec2.tf
├── s3.tf
├── security-group.tf
└── variables.tf
```

All resources in the same root module use the same configured backend and state.

## 10. Do we need to run terraform init every time?

No!

Once Terraform is initialized, we do not need to run `terraform init` after every resource change.

| Situation | Need terraform init? |
|---|---|
| Creating a new EC2 | No |
| Modifying an EC2 | No |
| Creating a state backup | No |
| Running terraform plan | No |
| Running terraform apply | No |
| Changing backend configuration | Yes |
| Adding or changing provider/module dependencies | Yes |

Important commands:

```bash
terraform init
terraform init -migrate-state
terraform init -reconfigure
terraform plan
terraform apply
```

### Difference between migrate-state and reconfigure

- `terraform init -migrate-state`: Migrates existing state to the newly configured backend.
- `terraform init -reconfigure`: Reinitializes the backend configuration without migrating the previous state.

## 11. Important Security Practices

1. Never upload Terraform state files to GitHub.
2. Never expose state files publicly.
3. Enable S3 versioning for recovery.
4. Enable encryption.
5. Configure proper IAM permissions.
6. Restrict access to authorized engineers.
7. Never hardcode AWS credentials in Terraform files.

## 12. Important Interview Questions and Answers

**Q1. Why do we store Terraform State remotely?**

We store Terraform State remotely to provide centralized state management and team collaboration. It allows authorized engineers to access the same infrastructure state.

**Q2. What is a Terraform Backend?**

A backend defines where Terraform stores its state file and how it manages state operations.

**Q3. What is the default backend in Terraform?**

The default backend is the Local Backend.

**Q4. Why do we use AWS S3 as a backend?**

We use AWS S3 to store Terraform state centrally and support team collaboration, versioning and access control.

**Q5. What is State Locking?**

State Locking prevents multiple Terraform operations from modifying the same state simultaneously.

**Q6. What is the purpose of `use_lockfile = true`?**

It enables S3 lockfile-based state locking.

**Q7. Why do we enable S3 Versioning?**

It helps recover previous versions of the state file if an unwanted update or deletion occurs.

**Q8. Can multiple engineers use the same Remote State?**

Yes. Multiple authorized engineers can access the same remote state using the shared backend configuration and appropriate permissions.

**Q9. Do we need a separate backend block for every resource?**

No. One backend configuration is sufficient for all resources managed by the same root module.

**Q10. Do we need to run terraform init every time?**

No. We initialize Terraform when setting up the backend or when backend configuration or required dependencies change.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Local State | Stored on local machine |
| Remote State | Stored in remote storage |
| Backend | Defines state storage |
| S3 Backend | Stores state in AWS S3 |
| State Locking | Prevents concurrent state modifications |
| S3 Versioning | Helps recover older versions |
| `terraform init` | Initializes backend |
| `-migrate-state` | Migrates existing state |
| `-reconfigure` | Reinitializes backend without migration |

## Golden Interview Answer

A Terraform backend defines where Terraform stores its state file. We use remote backends like AWS S3 to centralize state management, support team collaboration, enable state locking and provide better backup and recovery options.

**Remember:**

- Local State = Stored on an individual machine.
- Remote State = Stored centrally for authorized team access.
- Backend = Tells Terraform where to store the state.
- State Locking = Helps prevent concurrent state modifications.

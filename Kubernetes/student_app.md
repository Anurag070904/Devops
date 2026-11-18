# Student Registration Application Deployment on AWS EKS

## 1. Project Overview

### Project Name
Student Registration Application – Containerization and Kubernetes Deployment on AWS EKS

### Objective

The main objective of this project is to containerize a Student Registration application and deploy it on Amazon Elastic Kubernetes Service (EKS) using Kubernetes resources.

The project demonstrates how a modern application can be deployed using Docker, AWS ECR, Amazon EKS, Kubernetes Deployments, Services, ConfigMaps, Secrets, StatefulSets and Persistent Volumes.

The application consists of:

- Frontend: User interface for student registration.
- Backend: Spring Boot application that handles API requests.
- Database: MariaDB for storing student information.

The database is configured to use persistent AWS EBS storage so that database data can survive container restarts and Pod replacement.

---

## 2. Technologies Used

| Technology | Purpose |
|---|---|
| Docker | Containerization |
| AWS ECR | Store Docker images |
| AWS EKS | Managed Kubernetes cluster |
| Kubernetes | Container orchestration |
| Amazon EBS | Persistent database storage |
| MariaDB | Relational database |
| ConfigMap | Store non-sensitive configuration |
| Secret | Store database credentials |
| YAML | Define Kubernetes resources |
| Ubuntu EC2 | Management machine |

---

## 3. Application Architecture

The application contains three major components:

### Frontend
The frontend provides the user interface where users can register and manage student records.

- Runs inside a Docker container.
- Built using Node.js and Apache.
- Exposes port 80.
- Deployed using a Kubernetes Deployment.
- Exposed through a Kubernetes LoadBalancer Service.

### Backend
The backend handles business logic and database communication.

- Built using Spring Boot and Java.
- Packaged into a Docker image.
- Exposes port 8080.
- Deployed using a Kubernetes Deployment.
- Uses an internal ClusterIP Service.

### Database
MariaDB stores student registration information.

- Runs as a Kubernetes StatefulSet.
- Uses an AWS EBS-backed PersistentVolumeClaim.
- Database credentials are obtained from Kubernetes Secret.
- Database configuration is obtained from ConfigMap.

### Application Flow

1. User accesses the frontend.
2. Frontend communicates with backend APIs.
3. Backend processes the request.
4. Backend connects to MariaDB.
5. MariaDB stores or retrieves student information.
6. Backend returns the response to the frontend.

Note: The intended application flow is configured, but end-to-end runtime validation is still pending.

---

## 4. Project Implementation – Step by Step

### Step 1: Dockerize the Backend

Created a Dockerfile inside the backend directory.

The backend Dockerfile uses:
- Maven with Java 17.
- Copies application source code.
- Builds the Spring Boot application.
- Exposes port 8080.
- Starts the application using Java.

Docker build command:

```bash
docker build -t student-backend:v1 ./backend
```

### Step 2: Dockerize the Frontend

Created a Docker image for the frontend application.

The frontend Dockerfile:
- Uses Node.js Alpine image.
- Copies frontend source code.
- Installs dependencies.
- Builds the frontend application.
- Installs Apache.
- Copies the production build into Apache's web directory.
- Exposes port 80.

Build command:

```bash
docker build -t student-frontend:v1 ./frontend
```

Both Docker images were built successfully.

### Step 3: Create AWS ECR Repositories

Amazon Elastic Container Registry (ECR) is used to store Docker images.

Created two repositories:

```bash
aws ecr create-repository \
  --repository-name student-backend \
  --region ap-south-1

aws ecr create-repository \
  --repository-name student-frontend \
  --region ap-south-1
```

### Step 4: Authenticate Docker with AWS ECR

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin \
416113246991.dkr.ecr.ap-south-1.amazonaws.com
```

### Step 5: Tag Docker Images

```bash
docker tag student-backend:v1 \
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-backend:v1

docker tag student-frontend:v1 \
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-frontend:v1
```

### Step 6: Push Images to ECR

```bash
docker push \
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-backend:v1

docker push \
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-frontend:v1
```

Both images were successfully pushed to AWS ECR.

---

## 5. Amazon EKS Cluster

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service provided by AWS.

It manages the Kubernetes control plane, while worker nodes run application Pods.

Our cluster configuration:

| Property | Value |
|---|---|
| Cluster Name | student-eks-demo |
| Region | ap-south-1 |
| Kubernetes Version | 1.34 |
| Node Group | student-eks-nodes |
| Worker Instance | t3.micro |
| Desired Nodes | 1 |

Kubernetes configuration was connected using:

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name student-eks-demo
```

Verify the cluster:

```bash
kubectl get nodes
```

The worker node was successfully registered and showed Ready status.

---

## 6. Kubernetes Namespace

Created a dedicated namespace for the application.

```bash
kubectl create namespace student-app
```

Why use a Namespace?

- Separates application resources from system resources.
- Makes resource management easier.
- Helps organize deployments, services, secrets and configurations.

---

## 7. ConfigMap

A ConfigMap stores non-sensitive application configuration separately from container images.

Created the MariaDB ConfigMap:

```bash
kubectl create configmap mariadb-config \
--namespace student-app \
--from-literal=MARIADB_DATABASE=student_db \
--from-literal=MARIADB_USER=studentuser
```

Configuration stored:

| Key | Value |
|---|---|
| MARIADB_DATABASE | student_db |
| MARIADB_USER | studentuser |

Why ConfigMap?

It allows configuration to be changed without rebuilding Docker images.

---

## 8. Kubernetes Secret

A Secret is used to store sensitive information such as passwords.

Created a Secret interactively:

```bash
read -rsp "Enter MariaDB root password: " DB_ROOT_PASSWORD
echo

read -rsp "Enter MariaDB user password: " DB_USER_PASSWORD
echo

kubectl create secret generic mariadb-secret \
--namespace student-app \
--from-literal=MARIADB_ROOT_PASSWORD="$DB_ROOT_PASSWORD" \
--from-literal=MARIADB_PASSWORD="$DB_USER_PASSWORD"

unset DB_ROOT_PASSWORD DB_USER_PASSWORD
```

The passwords are not hardcoded in the MariaDB StatefulSet YAML.

Why Secret?

- Keeps credentials separate from application configuration.
- Allows Pods to consume passwords through environment variables.
- Avoids putting plaintext passwords directly in Deployment files.

Important: Kubernetes Secrets are base64-encoded by default, not automatically encrypted at rest in every configuration. Production environments should use appropriate encryption and access controls.

---

## 9. AWS EBS CSI Driver

Amazon EBS provides block storage that can be attached to EC2 instances.

To use EBS dynamically with Kubernetes, we configured the AWS EBS CSI Driver.

### OIDC Provider

Associated the cluster with an IAM OIDC provider:

```bash
eksctl utils associate-iam-oidc-provider \
--cluster student-eks-demo \
--region ap-south-1 \
--approve
```

### IAM Role

Created the IAM service-account role for the EBS CSI Driver with the required EBS CSI permissions.

### EBS CSI Add-on

Installed the EBS CSI add-on:

```bash
aws eks create-addon \
--cluster-name student-eks-demo \
--addon-name aws-ebs-csi-driver \
--service-account-role-arn arn:aws:iam::416113246991:role/AmazonEKS_EBS_CSI_DriverRole \
--region ap-south-1
```

Purpose:

The CSI driver allows Kubernetes to provision and manage AWS EBS volumes for workloads.

---

## 10. StorageClass

Created `mariadb-storage.yaml`.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
allowVolumeExpansion: true
parameters:
  type: gp3
```

Explanation:

- `provisioner`: Uses the AWS EBS CSI Driver.
- `volumeBindingMode`: Waits until a Pod is scheduled before provisioning the volume.
- `reclaimPolicy`: Delete means the dynamically provisioned volume can be deleted when its PVC is released.
- `allowVolumeExpansion`: Allows volume expansion.
- `type`: Uses AWS gp3 EBS storage.

---

## 11. MariaDB Headless Service

Created `mariadb-service.yaml`.

The Service is configured with:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: student-app
spec:
  clusterIP: None
  selector:
    app: mariadb
  ports:
    - name: mariadb
      port: 3306
      targetPort: 3306
```

Why Headless Service?

A headless Service does not allocate a normal ClusterIP. It provides stable network identity and DNS discovery for StatefulSet Pods.

The MariaDB service name is used by the backend for database connection:

```text
jdbc:mariadb://mariadb:3306/student_db
```

---

## 12. MariaDB StatefulSet

Created `mariadb-statefulset.yaml`.

A StatefulSet is a Kubernetes workload resource designed for applications that need stable identities and persistent storage.

Our MariaDB StatefulSet configuration includes:

- MariaDB image: mariadb:11.4
- Replicas: 1
- Container port: 3306
- ConfigMap-based database configuration.
- Secret-based credentials.
- PersistentVolumeClaim template.
- Mount path: /var/lib/mysql
- Storage request: 5Gi
- StorageClass: gp3

### Why StatefulSet instead of Deployment?

Databases need stable identity and persistent storage.

A StatefulSet provides:
- Stable Pod identity.
- Predictable Pod naming.
- PersistentVolumeClaim association.
- Ordered Pod management.

### Why Persistent Volume?

Container filesystems are temporary. If a container is recreated, data stored only inside its writable layer may be lost.

Persistent storage allows database data to survive container replacement and Pod recreation, provided the persistent volume is retained and managed correctly.

---

## 13. Backend Deployment and Service

Created `backend.yaml`.

The backend Deployment uses:

```text
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-backend:v1
```

Container port: 8080.

Database environment variables:

```text
SPRING_DATASOURCE_URL=jdbc:mariadb://mariadb:3306/student_db
SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD
```

The username comes from ConfigMap and password comes from Secret.

Backend Service:

- Type: ClusterIP
- Port: 8080
- Target Port: 8080

Why ClusterIP?

The backend is intended to be accessed internally by other Kubernetes workloads, rather than exposing the database-connected API service directly to the public internet.

---

## 14. Frontend Deployment and Service

Created `frontend.yaml`.

The frontend Deployment uses:

```text
416113246991.dkr.ecr.ap-south-1.amazonaws.com/student-frontend:v1
```

Container port: 80.

Frontend Service:

- Type: LoadBalancer
- Port: 80
- Target Port: 80

Why LoadBalancer?

A LoadBalancer Service provisions an external AWS load balancer so users can access the frontend through an external DNS address.

---

## 15. Combined Deployment YAML

We also prepared a `deployment.yaml` file containing:

1. Backend Deployment.
2. Backend Service.
3. Frontend Deployment.
4. Frontend Service.

This makes it easier to manage both application components from a single YAML file.

Apply command:

```bash
kubectl apply -f deployment.yaml
```

---

## 16. Kubernetes Deployment Commands

Apply the application resources:

```bash
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
```

Check Deployments:

```bash
kubectl get deployments -n student-app
```

Check Pods:

```bash
kubectl get pods -n student-app
```

Check Services:

```bash
kubectl get services -n student-app
```

Check StatefulSet:

```bash
kubectl get statefulsets -n student-app
```

Check PersistentVolumeClaims:

```bash
kubectl get pvc -n student-app
```

Check detailed Pod events:

```bash
kubectl describe pod <pod-name> -n student-app
```

---

## 17. Current Implementation Status and Troubleshooting

The Docker images were successfully built and pushed to AWS ECR.

The EKS cluster and worker node were created successfully.

Kubernetes resources were also successfully submitted to the cluster.

However, during runtime scheduling, Kubernetes reported:

```text
0/1 nodes are available: 1 Too many pods.
```

### Root Cause

The current worker node has a maximum Pod capacity of 4.

The existing system Pods already occupy the available slots:

- aws-node
- coredns
- ebs-csi-node
- kube-proxy

As a result:

- MariaDB Pod remains Pending.
- Backend Pod remains Pending.
- Frontend Pod remains Pending.
- EBS CSI controller Pods remain Pending.
- An additional CoreDNS Pod remains Pending.

The worker node has approximately 1 GB total memory, with approximately 500 MiB allocatable to Kubernetes.

### Resolution Required

The next step is to increase worker-node capacity using an eligible instance type and an appropriate EKS node-group configuration.

After sufficient capacity is available:

1. Verify that the new worker node is Ready.
2. Verify EBS CSI controller Pods are Running.
3. Verify the MariaDB PVC is Bound.
4. Verify MariaDB Pod is Running.
5. Verify backend and frontend Pods are Running.
6. Verify database connectivity.
7. Correct the frontend API URL and validate frontend-to-backend communication.
8. Test student registration functionality.

**Current result:** Infrastructure and Kubernetes configuration prepared; live application validation is pending because of worker-node Pod capacity.

---

## 18. Important Concepts for Trainer Viva

### What is Docker?
Docker is a containerization platform used to package an application and its dependencies into a portable container image.

### What is Kubernetes?
Kubernetes is an open-source container orchestration platform used to deploy, scale and manage containerized applications.

### What is Amazon EKS?
Amazon EKS is AWS's managed Kubernetes service.

### What is ECR?
Amazon Elastic Container Registry is a managed container image registry used to store Docker images.

### What is a Pod?
A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers.

### What is a Deployment?
A Deployment manages stateless application Pods and maintains the desired number of replicas.

### What is StatefulSet?
A StatefulSet manages stateful applications that require stable identities and persistent storage.

### What is a Service?
A Service provides a stable network endpoint for accessing a group of Pods.

### What is ClusterIP?
ClusterIP exposes a Service internally within the Kubernetes cluster.

### What is LoadBalancer?
LoadBalancer exposes a Service externally through a cloud-provider load balancer.

### What is ConfigMap?
A ConfigMap stores non-sensitive configuration separately from application images.

### What is Secret?
A Secret stores sensitive configuration such as passwords and tokens.

### What is PersistentVolume?
A PersistentVolume represents storage available to Kubernetes workloads.

### What is PersistentVolumeClaim?
A PersistentVolumeClaim is a request for storage made by a Kubernetes workload.

### Why EBS?
Amazon EBS provides persistent block storage that can be attached to EC2 instances and used by Kubernetes through the EBS CSI Driver.

### Why use MariaDB StatefulSet?
Because a database requires stable identity and persistent storage across Pod recreation.

---

## 19. Conclusion

This project demonstrates the end-to-end implementation approach for containerizing and deploying a Student Registration application on AWS EKS.

The project includes Docker image creation, image storage in Amazon ECR, Kubernetes cluster setup, application configuration, database credentials management, MariaDB StatefulSet configuration, and persistent AWS EBS storage integration.

The Kubernetes application resources have been created and applied. The remaining task is to resolve worker-node Pod capacity and complete live application validation.

The project provides practical experience with Docker, AWS cloud services, Kubernetes orchestration, persistent storage, and containerized application architecture.

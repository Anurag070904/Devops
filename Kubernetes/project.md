# Student App on AWS EKS – Simple Documentation

Nov 24, 2026 · @Anurag

## 1. What the project is

This project deploys a simple **Student Registration web app** on **AWS EKS** (managed Kubernetes). A user fills in a form in the browser, the app saves the student in a MariaDB database, and the student appears in a table.

The code is in one Git repo, `student-app-k8s`, with a `backend/` folder (Spring Boot, Java 17) and a `frontend/` folder (React with Vite).

**What the trainer asked us to use**

- **Deployment** for the frontend and the backend
- **StatefulSet** for the MariaDB database
- **ConfigMap** and **Secret** for database settings and passwords
- **PV** and **PVC** for the database disk
- **HPA** (autoscaler) for the frontend and the backend

**Where we are now**

- The EKS cluster `student-eks` was created in region `ap-south-1`.
- The first `eksctl create cluster` command seemed stuck, and the node group was missing. We found the cluster was fine and created the node group separately (see section 9).
- Next: confirm the nodes are `Ready`, then continue from Phase 1, step 4 below.

## 3. Key terms in simple words

| Term | Simple meaning | Used for |
| --- | --- | --- |
| Namespace | A folder that groups all our app's resources | `student-app` |
| Deployment | Runs app containers and restarts them if they crash | Frontend, backend |
| StatefulSet | Like a Deployment, but keeps a stable name and its own disk | MariaDB |
| Service | A fixed network address for a set of pods | One per component |
| LoadBalancer Service | A Service that gets a public internet address from AWS | Frontend, backend |
| Headless Service | A Service with no load balancing, just a stable DNS name (`clusterIP: None`) | MariaDB |
| ConfigMap | Stores normal settings | DB name, DB user, DB URL |
| Secret | Stores sensitive settings | DB passwords |
| StorageClass | The recipe for making disks (here: AWS EBS gp3) | `gp3` |
| PVC (PersistentVolumeClaim) | A request for a disk ("I need 20Gi") | Created by the StatefulSet |
| PV (PersistentVolume) | The real disk, an AWS EBS volume | Created automatically |
| HPA | Adds or removes pods when CPU goes up or down | Frontend, backend |

## 4. Phase 1: Setup (cluster and tools)

Everything is run from one Ubuntu EC2 instance, called the **main EC2**.

1. **Launch the main EC2.** Ubuntu, e.g. `c7i-flex.large`, 20 GB disk. Attach an IAM role with `AdministratorAccess`, then connect by SSH.
2. **Install the tools** (AWS CLI, kubectl, eksctl, Docker, git):

```bash
sudo apt update -y && sudo apt install -y curl unzip git docker.io
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip && sudo ./aws/install
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
sudo usermod -aG docker ubuntu && newgrp docker
```

Check with `aws --version`, `kubectl version --client`, `eksctl version`.

3. **Create the cluster** (about 15 to 20 minutes). Our cluster is named `student-eks`:

```bash
eksctl create cluster --name student-eks --region ap-south-1 \
  --nodegroup-name linux-nodes --node-type m7i-flex.large --nodes 2
```

4. **Create the node group separately** (needed if the command above stalls or no node group appears):

```bash
aws eks update-kubeconfig --name student-eks --region ap-south-1
eksctl create nodegroup --cluster student-eks --region ap-south-1 \
  --name linux-nodes --node-type m7i-flex.large --nodes 2 --managed
kubectl get nodes
```

You should see 2 nodes with status `Ready`.

5. **Add the EBS CSI driver** so Kubernetes can create disks. Create an IAM role with `AmazonEBSCSIDriverPolicy`. In the EKS console open the cluster, go to **Add-ons**, add **Amazon EBS CSI Driver** and attach that role. Wait for **Active**.
6. **Install metrics-server**, which the HPA needs to read CPU:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

7. **Clone the repo:**

```bash
git clone https://github.com/prathamnandgirwar/student-app-k8s.git
cd student-app-k8s
```

## 5. Phase 2: Database (StatefulSet, ConfigMap, Secret, PVC and PV)

MariaDB runs inside Kubernetes. Pods reach it by the Service name `mariadb`, so no worker-EC2 database is needed. The passwords below are demo values; change them.

1. **Create the namespace:** `kubectl create namespace student-app`
2. **StorageClass** (`gp3-storageclass.yaml`):

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

3. **ConfigMap and Secret** (`mariadb-config.yaml`, `mariadb-secret.yaml`):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mariadb-config
  namespace: student-app
data:
  MARIADB_DATABASE: student_db
  MARIADB_USER: admin
  DB_URL: jdbc:mariadb://mariadb:3306/student_db
---
apiVersion: v1
kind: Secret
metadata:
  name: mariadb-secret
  namespace: student-app
type: Opaque
stringData:
  MARIADB_ROOT_PASSWORD: "ChangeMe123"
  MARIADB_PASSWORD: "ChangeMe123"
```

4. **Headless Service** (`mariadb-service.yaml`):

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
    - port: 3306
      targetPort: 3306
```

5. **StatefulSet** (`mariadb-statefulset.yaml`). The `volumeClaimTemplates` part creates the PVC, and the EBS driver then creates the PV:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
  namespace: student-app
spec:
  serviceName: mariadb
  replicas: 1
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
        - name: mariadb
          image: mariadb:11.4
          ports:
            - containerPort: 3306
          envFrom:
            - configMapRef:
                name: mariadb-config
            - secretRef:
                name: mariadb-secret
          volumeMounts:
            - name: mariadb-storage
              mountPath: /var/lib/mysql
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
            limits:   { cpu: "1000m", memory: "1Gi" }
  volumeClaimTemplates:
    - metadata:
        name: mariadb-storage
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp3
        resources:
          requests:
            storage: 20Gi
```

6. **Apply and verify:**

```bash
kubectl apply -f gp3-storageclass.yaml
kubectl apply -f mariadb-config.yaml -f mariadb-secret.yaml -f mariadb-service.yaml -f mariadb-statefulset.yaml
kubectl get pods -n student-app    # mariadb-0 should be 1/1 Running
kubectl get pvc -n student-app     # should be Bound
kubectl get pv                     # the auto-created disk
```

The PVC shows `Pending` until the pod starts. That is normal because of `WaitForFirstConsumer`.

## 6. Phase 3: Backend (Deployment, Service, HPA)

The repo's `application.properties` has hard-coded RDS details. Spring Boot lets environment variables override it, so we feed the values from the ConfigMap and Secret and leave the file alone.

1. **Build and push the image** (needs a free Docker Hub account):

```bash
cd backend
docker build -t <your-dockerhub-user>/backend .
docker login -u <your-dockerhub-user>
docker push <your-dockerhub-user>/backend
cd ..
```

2. **`backend.yaml`** (Deployment and LoadBalancer Service):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deploy
  namespace: student-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: <your-dockerhub-user>/backend
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_DATASOURCE_URL
              valueFrom:
                configMapKeyRef: { name: mariadb-config, key: DB_URL }
            - name: SPRING_DATASOURCE_USERNAME
              valueFrom:
                configMapKeyRef: { name: mariadb-config, key: MARIADB_USER }
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef: { name: mariadb-secret, key: MARIADB_PASSWORD }
          resources:
            requests: { cpu: "100m", memory: "256Mi" }
            limits:   { cpu: "500m", memory: "512Mi" }
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: student-app
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - port: 8080
      targetPort: 8080
```

3. **`backend-hpa.yaml`.** In plain words: keep 2 to 5 pods, and add more when average CPU goes above 60%.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: student-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deploy
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

4. **Apply and verify:**

```bash
kubectl apply -f backend.yaml -f backend-hpa.yaml
kubectl get pods -n student-app
kubectl get svc -n student-app      # copy the EXTERNAL-IP of backend-svc
kubectl get hpa -n student-app
kubectl logs deploy/backend-deploy -n student-app
```

The `EXTERNAL-IP` hostname can take 2 to 3 minutes to appear. The frontend needs it in the next phase.

## 7. Phase 4: Frontend (Deployment, Service, HPA)

The frontend runs in the user's browser, so it needs the backend's **public** address. That address is baked into the image at build time, so set it before building.

1. **Point the frontend at the backend:**

```bash
cd frontend
echo "VITE_API_URL=http://<backend-EXTERNAL-IP>:8080/api" > .env
```

2. **Build and push the image:**

```bash
docker build -t <your-dockerhub-user>/frontend .
docker push <your-dockerhub-user>/frontend
cd ..
```

3. **`frontend.yaml` and `frontend-hpa.yaml`** follow the same pattern as the backend:

| File | Resource | Key settings |
| --- | --- | --- |
| `frontend.yaml` | Deployment `frontend-deploy` | image `<your-dockerhub-user>/frontend`, container port 80, 1 replica, CPU and memory requests and limits |
| `frontend.yaml` | Service `frontend-svc` | type `LoadBalancer`, port 80 to 80 |
| `frontend-hpa.yaml` | HPA `frontend-hpa` | targets `frontend-deploy`, 1 to 3 pods, CPU 60% |

4. **Apply and get the public address:**

```bash
kubectl apply -f frontend.yaml -f frontend-hpa.yaml
kubectl get svc -n student-app      # copy the EXTERNAL-IP of frontend-svc
kubectl get hpa -n student-app
```

## 8. Phase 5: Test everything

1. **Open the app.** Go to `http://<frontend-EXTERNAL-IP>` in a browser. The registration form should load with an empty table.
2. **Register a student.** Fill in the form and click **Register**. The student should appear in the table.
3. **Check the database:**

```bash
kubectl exec -it mariadb-0 -n student-app -- mariadb -u admin -p student_db -e "SHOW TABLES;"
```

4. **Prove the data survives a restart** (the point of the StatefulSet and PVC):

```bash
kubectl delete pod mariadb-0 -n student-app
kubectl get pods -n student-app -w
```

A new `mariadb-0` starts and attaches to the same disk, so the student is still in the table after you refresh the page.

## 9. Problems we hit and how we fixed them

| Problem | What we saw | Fix |
| --- | --- | --- |
| Cluster creation looked stuck | The terminal kept printing "waiting for CloudFormation stack" for about 30 minutes | Check the CloudFormation console. The stack `eksctl-student-eks-cluster` already showed `CREATE_COMPLETE`, so the terminal had frozen. Open a new SSH session. Run long commands inside `tmux` so they survive a dropped connection. |
| No node group | No `...-nodegroup-linux-nodes` stack existed after the cluster stack finished | Create it separately with `eksctl create nodegroup` (Phase 1, step 4) |
| `ResourceNotFoundException: No cluster found for name: student-eks-cluster` | `eksctl get cluster` listed the name `student-eks` | The stack name is `eksctl-<name>-cluster`, so the real cluster name is `student-eks`. Always copy the name from `eksctl get cluster --region ap-south-1`. |
| Earlier node group failed | Stack `student-app-cluster-nodegroup-student-app-nodes` ended in `ROLLBACK_COMPLETE` on 18 Nov | Open the stack's **Events** tab and read the red Status reason. If it is an instance-type problem, retry with `--node-type t3.medium`. |
| Many old clusters | Four EKS cluster stacks in one region | Each control plane costs about $0.10 per hour, and VPC and Elastic IP quotas can run out. Delete clusters you no longer use. |
| Pod stuck `Pending` | `mariadb-0` never starts | The EBS CSI add-on is not Active or its IAM role is missing. Check with `kubectl describe pod mariadb-0 -n student-app`. |
| Backend `CrashLoopBackOff` | Backend pods keep restarting | The database is not ready yet, or the Secret password does not match. Read `kubectl logs deploy/backend-deploy -n student-app`. |
| HPA shows `<unknown>` | CPU target cannot be read | metrics-server is not installed, or the pods have no CPU `requests`. |
| Page loads but the table is empty or errors | Frontend cannot reach the backend | `VITE_API_URL` was wrong when the image was built. Fix `.env`, rebuild and push, then `kubectl rollout restart deploy/frontend-deploy -n student-app`. |

## 10. Cleanup and command cheat sheet

**Cleanup.** AWS charges while the cluster runs (about $0.10 per hour for the control plane, plus nodes and load balancers). Delete everything when you finish:

```bash
kubectl delete namespace student-app      # removes the load balancers first
eksctl delete cluster --name student-eks --region ap-south-1
```

The StorageClass uses `Retain`, so the EBS volume is kept. Delete it by hand in the EC2 console under **Volumes** to stop paying for it.

**Cheat sheet**

| Command | What it does |
| --- | --- |
| `kubectl get nodes` | Lists worker nodes and their status |
| `kubectl get pods -n student-app` | Lists app pods |
| `kubectl get svc -n student-app` | Shows services and public addresses |
| `kubectl get pvc -n student-app` / `kubectl get pv` | Shows the disk claim and the disk |
| `kubectl get hpa -n student-app` | Shows autoscaler status |
| `kubectl describe pod <name> -n student-app` | Explains why a pod is failing |
| `kubectl logs <pod> -n student-app` | Shows a pod's logs |
| `kubectl rollout restart deploy/<name> -n student-app` | Restarts a Deployment with the latest image |
| `eksctl get cluster --region ap-south-1` | Lists clusters (and their real names) |

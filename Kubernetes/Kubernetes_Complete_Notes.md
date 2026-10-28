# Kubernetes (K8s) --- Complete DevOps Notes

**Beginner to production concepts \| Simple English + Indian real-world
examples \| YAML practice**

> **Learning goal:** Understand what each Kubernetes object does, why it
> exists, how to write its YAML, how objects connect, and what to check
> when something fails.
>
> **Important:** YAML examples are learning templates. Replace image
> names, domains, storage classes, resource sizes, AWS settings, and
> credentials for your environment. Some examples require cluster
> add-ons (Ingress controller, metrics-server, CSI storage driver, or an
> operator). Never put real passwords in Git.

## Table of contents

1.  What Kubernetes is and why we use it
2.  Architecture
3.  YAML basics and kubectl
4.  Pods
5.  Amazon EKS
6.  Labels and selectors: equality-based vs set-based
7.  Namespaces
8.  ReplicaSet and ReplicationController
9.  Deployments and deployment strategies
10. Services and service YAML
11. Intra-Pod and inter-Pod communication
12. ConfigMap and Secret
13. Volumes, PV and PVC
14. StatefulSet
15. DaemonSet
16. Operators
17. Autoscaling: HPA, VPA and Cluster Autoscaler
18. Ingress
19. How everything connects
20. Useful commands and troubleshooting
21. Production checklist and interview revision
22. Official references

------------------------------------------------------------------------

## 1. What is Kubernetes (K8s)?

### Definition (interview-ready)

**Kubernetes is an open-source container orchestration platform that
automates deploying, scaling, networking, and managing containerized
applications.**

Docker can build and run a container. Kubernetes coordinates application
containers across a group of machines called a **cluster**.

### Why do we use it?

Imagine an online shopping application has a Spring Boot API. During a
sale, traffic increases sharply. A single container can become
overloaded or crash. In a production environment we want to: - Run
multiple copies of the application. - Replace failed Pods
automatically. - Distribute requests to healthy Pods. - Roll out a new
version gradually and roll back if needed. - Keep configuration separate
from the image. - Attach persistent storage where required. - Scale Pods
and, when configured, the worker nodes. - Apply access controls and
isolate teams/environments.

**Real-world example (Marathi/Hindi):** Samja ek food-delivery app aahe.
Lunch time la orders khup vadhatat. Kubernetes extra application Pods
start karayla madat karto; traffic kami zala ki replicas kami karta
yetat. Pan autoscaling sathi proper metrics, requests, permissions and
autoscaler configuration pan lagte---Kubernetes magic ne unlimited
servers create karat nahi.

### Kubernetes does not automatically solve everything

-   It does not make buggy code correct.
-   It does not guarantee zero downtime without correct readiness
    probes, replicas, capacity, and rollout configuration.
-   It does not automatically back up your database.
-   It does not mean every workload should be containerized.
-   It needs monitoring, logging, security, networking, and storage
    design.

------------------------------------------------------------------------

## 2. Kubernetes architecture

A Kubernetes cluster has two broad parts: **control plane** and **worker
nodes**.

### 2.1 Control plane --- the manager

-   **kube-apiserver:** The front door/API. `kubectl` and controllers
    talk to the API server.
-   **etcd:** Key-value store holding Kubernetes cluster state and
    configuration. Protect and back it up according to your platform's
    operating model.
-   **kube-scheduler:** Chooses a suitable node for a newly created Pod
    based on resource requests, constraints, affinity,
    taints/tolerations, and other scheduling rules.
-   **kube-controller-manager:** Runs controllers that compare actual
    state with desired state and try to correct differences.
-   **cloud-controller-manager:** Integrates Kubernetes with a cloud
    provider where applicable, for example cloud load balancers or
    routes.

### 2.2 Worker node --- where application Pods run

-   **kubelet:** Node agent that makes sure assigned Pods/containers are
    running as requested.
-   **Container runtime:** Runs containers, for example containerd.
-   **kube-proxy or equivalent networking implementation:** Implements
    Service networking behavior in many clusters. Some networking
    solutions replace kube-proxy.
-   **Pods:** The workload units containing one or more containers.

### 2.3 How a Deployment request flows

1.  Engineer applies a YAML file using `kubectl apply -f app.yaml`.
2.  `kubectl` sends the desired object to the API server.
3.  The API server validates the request and persists state through
    etcd.
4.  Controllers notice that the requested Deployment/replicas need to be
    created and create the required ReplicaSet and Pods.
5.  The scheduler assigns unscheduled Pods to nodes.
6.  The kubelet on each chosen node asks the container runtime to run
    the containers.
7.  Readiness checks and Service endpoint management determine which
    Pods should receive traffic.
8.  Controllers continue reconciling the cluster toward the declared
    desired state.

**Key idea:** Kubernetes is declarative. You describe the desired state
(for example, three replicas), and controllers continuously try to make
reality match it.

------------------------------------------------------------------------

## 3. YAML basics and kubectl

Most Kubernetes manifest files use these fields: - `apiVersion`: API
group/version for the resource. - `kind`: Type of object, such as
`Deployment` or `Service`. - `metadata`: Name, namespace, labels and
annotations. - `spec`: Desired configuration. - `status`: Observed state
written by Kubernetes; usually do not set it in your manifest.

Example:

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Useful commands:

``` bash
kubectl apply -f hello-pod.yaml
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod
kubectl delete -f hello-pod.yaml
kubectl get all -A
kubectl explain deployment.spec
```

`containerPort` documents the port the container listens on; by itself
it does not publish the app to the internet.

------------------------------------------------------------------------

## 4. Pod

### What is a Pod?

A **Pod is the smallest deployable unit in Kubernetes**. It contains one
or more tightly related containers. Containers in the same Pod share the
Pod's network namespace (including IP and port space) and can share
mounted volumes.

Most Pods contain one application container. A sidecar pattern may put a
tightly coupled helper beside the app.

### Why not create containers directly?

Kubernetes schedules and manages Pods as the workload unit. A Pod has a
lifecycle and network identity; if it is replaced, the replacement
generally gets a new IP. For ordinary applications, create Pods through
a controller such as a Deployment rather than managing individual Pods
manually.

### Pod YAML

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-web
  labels:
    app: demo-web
spec:
  containers:
    - name: web
      image: nginx:1.27
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "500m"
          memory: "128Mi"
```

**Why these fields?** - `labels` allow a Service or other selector to
find the Pod. - `image` identifies the container image. -
`resources.requests` help the scheduler find a node with enough
capacity. - `resources.limits` constrain resource use (CPU is throttled
at a limit; memory overuse can cause an OOM kill).

Practice:

``` bash
kubectl apply -f pod.yaml
kubectl get pod demo-web -o wide
kubectl describe pod demo-web
kubectl delete pod demo-web
```

------------------------------------------------------------------------

## 5. Amazon EKS

### What is EKS?

**Amazon Elastic Kubernetes Service (EKS) is AWS's managed Kubernetes
service.** AWS manages the Kubernetes control plane; you still choose
and manage the worker/data-plane approach, workloads, permissions,
networking and operational practices according to your EKS setup.

### Why use EKS?

-   Managed, highly available Kubernetes control plane.
-   Integration with AWS networking, identity, load balancing, logging
    and storage.
-   Run workloads on EC2 worker nodes or supported serverless compute
    options such as Fargate.
-   Use standard Kubernetes APIs and tools.

### Simplified EKS architecture

-   **AWS-managed control plane:** API server and etcd are operated by
    AWS.
-   **Data plane:** EC2 nodes, Fargate Pods, or supported EKS Auto
    Mode-managed infrastructure.
-   **VPC and networking:** Subnets, security groups, Pod networking and
    routing must be designed correctly.
-   **IAM and Kubernetes RBAC:** AWS identity/access and Kubernetes
    authorization work together in the access model.
-   **ECR:** Common AWS container image registry.
-   **AWS Load Balancer Controller:** Common option to provision AWS
    load balancers from Kubernetes resources.
-   **EBS/EFS CSI drivers:** Common ways to provide persistent storage
    to Pods.

### Common EKS commands

``` bash
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
```

These commands assume AWS CLI credentials, permissions, and the cluster
already exist. The region and cluster name are examples.
`update-kubeconfig` configures local `kubectl` access; it does not
create the cluster.

### EKS gotchas

-   Creating a Kubernetes `Service` of type `LoadBalancer` may provision
    an AWS load balancer only when the cluster has the appropriate cloud
    integration/controller and permissions.
-   A PVC needs a compatible StorageClass/CSI driver.
-   Pod autoscaling requires metrics and an autoscaler; node autoscaling
    needs a supported node-scaling solution and correct permissions.
-   Never put AWS access keys in a ConfigMap, Secret manifest committed
    to Git, or container image. Prefer IAM roles for workloads and
    short-lived credentials.

------------------------------------------------------------------------

## 6. Labels and selectors

A **label** is a key/value tag on an object. A **selector** matches
objects based on labels. Services, ReplicaSets, Deployments, and many
other resources use selectors.

Example label:

``` yaml
labels:
  app: payment
  tier: backend
  environment: production
```

### Equality-based selector

Matches exact equality or inequality:

``` yaml
selector:
  matchLabels:
    app: payment
    environment: production
```

This matches objects with both exact label values. Multiple
`matchLabels` entries are ANDed together.

### Set-based selector

Supports set expressions such as `In`, `NotIn`, `Exists`, and
`DoesNotExist`:

``` yaml
selector:
  matchExpressions:
    - key: environment
      operator: In
      values: ["staging", "production"]
    - key: tier
      operator: Exists
```

This matches objects where `environment` is staging or production
**and** the `tier` label exists.

For a simple Service, `spec.selector` is normally a map of exact label
matches:

``` yaml
spec:
  selector:
    app: payment
```

Do not confuse Service selectors with the richer `matchExpressions`
selector format supported by resources such as Deployments and
ReplicaSets.

**Common mistake:** If a Service selector does not match the Pods'
labels, the Service can exist but have no usable backends.

------------------------------------------------------------------------

## 7. Namespace

### What is a Namespace?

A Namespace is a logical boundary for grouping namespaced resources in
one cluster. It is useful for team/environment separation, quotas, and
access policies. It is not a strong security boundary by itself.

Typical examples: `dev`, `staging`, `production`, `monitoring`.

### Namespace YAML

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Create an app in that namespace:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Commands:

``` bash
kubectl apply -f namespace.yaml
kubectl get pods -n dev
kubectl config set-context --current --namespace=dev
```

Some resources are cluster-scoped, not namespaced (for example Nodes and
PersistentVolumes).

------------------------------------------------------------------------

## 8. ReplicationController and ReplicaSet

### Why do they exist?

A controller maintains a desired number of matching Pods. If you request
three and one disappears, the controller tries to create a replacement.

### ReplicationController (legacy)

ReplicationController is an older Kubernetes replication resource. New
work generally uses ReplicaSets, usually managed by Deployments. Learn
it for legacy clusters and interviews, but do not choose it for a new
app.

Legacy-style example:

``` yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: old-web
spec:
  replicas: 2
  selector:
    app: old-web
  template:
    metadata:
      labels:
        app: old-web
    spec:
      containers:
        - name: web
          image: nginx:1.27
```

### ReplicaSet

A ReplicaSet maintains the desired number of Pods that match its
selector. It supports richer selectors. In most applications, let a
Deployment manage the ReplicaSet so you get rollout history and easier
updates.

``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-rs
  template:
    metadata:
      labels:
        app: web-rs
    spec:
      containers:
        - name: web
          image: nginx:1.27
```

### Difference

-   ReplicationController: older API and simpler selector behavior.
-   ReplicaSet: newer controller with set-based selector support.
-   Deployment: higher-level resource that manages ReplicaSets and
    rollout/rollback.

**Rule of thumb:** In modern projects, create a Deployment, not a
standalone ReplicaSet, unless you have a specific reason.

------------------------------------------------------------------------

## 9. Deployment

### What is a Deployment?

A Deployment manages a stateless application and declares its desired
Pod replicas and update strategy. It creates/manages ReplicaSets, which
manage Pods.

### Deployment YAML

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-api
  labels:
    app: spring-api
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: spring-api
  template:
    metadata:
      labels:
        app: spring-api
    spec:
      containers:
        - name: api
          image: nginx:1.27
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

The image is NGINX for a runnable example. Replace it with your
versioned application image. For Spring Boot, use the port and health
endpoint your app actually exposes.

### Why important fields?

-   `replicas: 3`: asks for three Pods.
-   `selector`: identifies the Pods managed by this Deployment.
-   `template`: specification used to create new Pods; template labels
    must match selector.
-   `RollingUpdate`: gradually replaces old Pods with new Pods.
-   `maxSurge: 1`: allows one extra Pod above the desired count during
    rollout.
-   `maxUnavailable: 0`: asks Kubernetes not to reduce the number of
    available replicas during rollout; sufficient capacity and healthy
    probes are still required.
-   `readinessProbe`: determines whether a Pod is ready to receive
    Service traffic.
-   `resources`: requests and limits CPU/memory.

Commands:

``` bash
kubectl apply -f deployment.yaml
kubectl get deploy,rs,pods
kubectl rollout status deployment/spring-api
kubectl set image deployment/spring-api api=your-registry/spring-api:v2
kubectl rollout history deployment/spring-api
kubectl rollout undo deployment/spring-api
kubectl scale deployment/spring-api --replicas=5
```

### Deployment strategies

1.  **Recreate:** Old Pods are terminated before new Pods are created.
    Simpler, but causes downtime for many apps.
2.  **RollingUpdate:** Gradually replaces old Pods with new Pods. The
    common Deployment default. Requires enough capacity and appropriate
    readiness checks.
3.  **Blue-Green:** Operate separate blue (current) and green (new)
    versions; switch traffic by changing a Service selector or routing
    configuration. This is a release pattern you implement with
    Kubernetes objects/tooling, not a built-in Deployment
    `strategy.type`.
4.  **Canary:** Send a small portion of traffic to a new version,
    validate it, then increase traffic. Usually implemented with an
    Ingress controller, service mesh, or progressive delivery tool; not
    a native Deployment strategy type by itself.

Example for Recreate:

``` yaml
strategy:
  type: Recreate
```

For normal production web APIs, RollingUpdate is a good starting point,
but deployment strategy depends on application compatibility, database
migrations, and availability requirements.

------------------------------------------------------------------------

## 10. Service and Service YAML

### What is a Service?

Pods are replaceable and their IP addresses can change. A **Service
provides a stable network endpoint and DNS name for a set of matching
Pods**. Kubernetes uses the Service selector to find the backend Pods
and routes traffic to eligible endpoints.

### Common Service types

-   **ClusterIP:** Internal access within the cluster; default.
-   **NodePort:** Exposes a port on cluster nodes; often used for
    learning or special network designs.
-   **LoadBalancer:** Requests an external/cloud load balancer when
    supported by the cluster integration.
-   **ExternalName:** Maps the Service DNS name to an external DNS name;
    it does not proxy traffic like the other types.

### ClusterIP Service YAML

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: spring-api
spec:
  type: ClusterIP
  selector:
    app: spring-api
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```

-   `port: 80`: port exposed by the Service.
-   `targetPort: 8080`: port on the selected Pod/container that receives
    traffic.
-   `selector`: must match the application Pod labels.

For the Deployment above, make the container listen on 8080 and label
its Pods `app: spring-api` before using this exact Service example.

### LoadBalancer Service example

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: public-api
spec:
  type: LoadBalancer
  selector:
    app: spring-api
  ports:
    - port: 80
      targetPort: 8080
```

This is not guaranteed to create a public endpoint on every cluster.
Cloud integration, permissions, subnet configuration, and controller
support matter. Use the platform's security requirements to decide
whether the load balancer should be public or internal.

### DNS

A Service in namespace `dev` is usually discoverable inside the cluster
by a name like: `spring-api.dev.svc.cluster.local` Short names often
work within the same namespace, such as `http://spring-api`.

------------------------------------------------------------------------

## 11. Intra-Pod and inter-Pod communication

### Intra-Pod communication

Containers in the same Pod share the network namespace and Pod IP. They
can talk through `localhost`, provided they use different ports where
necessary.

Example: an application container listens on `localhost:8080`; a tightly
coupled proxy sidecar can communicate with it through `127.0.0.1:8080`.
Containers in one Pod must avoid port conflicts because they share the
same port space.

### Inter-Pod communication

Pods can communicate over the cluster network using Pod IPs, subject to
the CNI network implementation and NetworkPolicies. Pod IPs are not
stable across replacement, so applications should normally use a Service
DNS name rather than hard-coding a Pod IP.

Typical path:
`frontend Pod -> backend Service DNS -> selected backend Pod`

Example:

``` yaml
# Frontend app configuration example
env:
  - name: BACKEND_URL
    value: "http://spring-api:80"
```

If frontend and backend are in different namespaces, use a qualified
name such as `http://spring-api.dev.svc.cluster.local`.

### Debugging connectivity

``` bash
kubectl get pods -o wide
kubectl get svc
kubectl get endpointslices
kubectl describe svc spring-api
kubectl exec -it <pod-name> -- sh
```

Check DNS, Service selector, EndpointSlices, port/targetPort,
application bind address, NetworkPolicies, and cloud/firewall rules.

------------------------------------------------------------------------

## 12. ConfigMap and Secret

### ConfigMap

Stores **non-sensitive configuration** separately from an image, such as
environment name, feature flags, or a non-secret URL.

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-config
data:
  APP_ENV: "production"
  LOG_LEVEL: "INFO"
  FEATURE_CHECKOUT: "true"
```

Use it as environment variables:

``` yaml
envFrom:
  - configMapRef:
      name: spring-config
```

Or mount ConfigMap data as files. Environment variables injected at
container start do not automatically update inside a running process
when the ConfigMap changes; mounted files can update eventually, but the
application must re-read them. A rollout restart may be needed.

### Secret

Stores sensitive values such as passwords, tokens, and keys. Kubernetes
Secret data is base64-encoded in the manifest/API representation;
**base64 is not encryption**. Configure encryption at rest,
least-privilege RBAC, audit controls, and preferably an external secret
manager for production.

Illustrative YAML only---do not commit real values:

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
stringData:
  username: "app-user"
  password: "replace-me-do-not-use-this-password"
```

`stringData` lets you provide plain strings and Kubernetes converts them
to stored Secret data. Use a secure creation/delivery workflow rather
than committing actual credentials.

Reference it in a Pod:

``` yaml
env:
  - name: DB_USERNAME
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: username
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: password
```

### ConfigMap vs Secret

-   ConfigMap: ordinary, non-sensitive configuration.
-   Secret: sensitive data; access must be restricted and storage
    secured.
-   Neither should be treated as a substitute for a proper
    secret-management process.

------------------------------------------------------------------------

## 13. Volumes, PersistentVolume (PV), and PersistentVolumeClaim (PVC)

### Why storage is needed

A container's writable layer is not the right place for important data
that must survive Pod replacement. Kubernetes Volumes make storage
available to containers.

### Volume

A Pod volume is mounted into one or more containers. Some volume types
are temporary; others refer to persistent storage. `emptyDir` lasts for
the life of the Pod, not forever.

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: cache-demo
spec:
  containers:
    - name: writer
      image: busybox:1.36
      command: ["sh", "-c", "date >> /cache/time.txt; sleep 3600"]
      volumeMounts:
        - name: cache
          mountPath: /cache
  volumes:
    - name: cache
      emptyDir: {}
```

### PV and PVC

-   **PersistentVolume (PV):** A cluster storage resource, provisioned
    by an administrator or dynamically by a storage provisioner.
-   **PersistentVolumeClaim (PVC):** A user's request for storage
    capacity and access mode.
-   **StorageClass:** Describes how storage can be dynamically
    provisioned, for example a class backed by an AWS EBS CSI driver.

In cloud clusters, dynamic provisioning through a StorageClass is
common. The exact StorageClass name depends on the cluster.

### PVC example

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  # Uncomment and set a StorageClass that exists in your cluster:
  # storageClassName: gp3
```

`ReadWriteOnce` generally means a volume can be mounted read-write by a
single node at a time; it does not always mean only one Pod can use it.
The storage driver and volume mode determine details. For multi-node
shared access, the backend must support an appropriate mode such as
`ReadWriteMany`.

### Mount PVC in a Pod

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-demo
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "echo hello >> /data/hello.txt; sleep 3600"]
      volumeMounts:
        - name: app-storage
          mountPath: /data
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: app-data
```

**Important:** A PVC may stay `Pending` if no matching
PV/StorageClass/provisioner exists, capacity is unavailable, or topology
constraints cannot be met. Persistent storage is not the same as a
backup; arrange backups and recovery testing separately.

------------------------------------------------------------------------

## 14. StatefulSet

### What is a StatefulSet?

A StatefulSet manages Pods that need stable identities and often stable
persistent storage. Pod names are predictable (`database-0`,
`database-1`, ...), and each replica can have its own PVC. StatefulSet
does not itself turn a database into a replicated/clustered database;
database replication and consistency must be configured by the database
software or an operator.

Use cases: databases, distributed queues, and systems where stable
identity/order matters. Use a Deployment for most stateless web/API
services.

### StatefulSet YAML (template)

This example requires a StorageClass that exists in your cluster:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web-stateful
  ports:
    - name: http
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web-stateful
spec:
  serviceName: web-headless
  replicas: 2
  selector:
    matchLabels:
      app: web-stateful
  template:
    metadata:
      labels:
        app: web-stateful
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 2Gi
        # Add storageClassName if your cluster needs a specific class.
```

-   Headless Service (`clusterIP: None`) provides stable network
    identities for StatefulSet Pods.
-   `serviceName` associates the StatefulSet with that governing
    Service.
-   `volumeClaimTemplates` creates a separate PVC for each Pod.
-   Pod identity is stable across replacement; the underlying node or IP
    may change.
-   StatefulSet Pods are typically created and terminated in an ordered
    manner by default; policy options can alter behavior.

Commands:

``` bash
kubectl get statefulset
kubectl get pods
kubectl get pvc
```

------------------------------------------------------------------------

## 15. DaemonSet

### What is a DaemonSet?

A DaemonSet ensures that a copy of a Pod runs on each eligible node (or
a selected subset of nodes). When eligible nodes are added, it schedules
Pods for them.

Use cases: - Log collection agents. - Node monitoring agents. -
Network/storage components. - Security agents that must run on nodes.

### DaemonSet YAML

``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      containers:
        - name: agent
          image: busybox:1.36
          command: ["sh", "-c", "while true; do sleep 3600; done"]
          resources:
            requests:
              cpu: "10m"
              memory: "16Mi"
```

This is only a harmless demonstration workload, not a real log
collector. Real node agents may require host mounts, permissions,
security configuration, and vendor-specific images. Do not grant
privileged access without a clear need.

### Compare workload controllers

-   Deployment: interchangeable stateless replicas and rollouts.
-   StatefulSet: stable Pod identity and per-replica storage.
-   DaemonSet: one Pod per eligible node.
-   Job/CronJob: finite or scheduled tasks (not covered deeply in this
    guide).

------------------------------------------------------------------------

## 16. Operators

### What is an Operator?

An Operator extends Kubernetes by combining a **Custom Resource
Definition (CRD)** with a controller that understands how to operate a
particular application. It automates domain-specific tasks such as
installation, configuration, upgrades, backups, failover, and
recovery---depending on what the operator is designed to support.

Example: a database operator may let you declare a database cluster
using a custom resource, then manage database Pods, storage, and
replication.

### Conceptual flow

1.  Install the operator and its CRD.
2.  Create a custom resource (CR) describing the desired database/app.
3.  The operator controller watches that resource.
4.  It reconciles related Kubernetes resources and application-specific
    state.
5.  It reports status and attempts to correct differences.

### Illustrative custom resource

This is **not runnable unless the named operator and its CRD are
installed**:

``` yaml
apiVersion: databases.example.com/v1
kind: DatabaseCluster
metadata:
  name: orders-db
spec:
  replicas: 3
  storage:
    size: 20Gi
```

`databases.example.com/v1`, `DatabaseCluster`, and fields are
placeholders, not a real standard API. Always use the CRD schema and
examples from the specific operator's documentation.

**Operator vs Deployment:** A Deployment understands how to keep a
number of Pods running. An application-specific Operator can understand
higher-level operations such as safe database upgrades or leader
election, if its implementation supports them.

------------------------------------------------------------------------

## 17. Autoscaling: HPA, VPA and CA

Autoscaling has different layers. **Pod scaling and node scaling are not
the same thing.**

### HPA --- Horizontal Pod Autoscaler

HPA changes the number of Pod replicas based on metrics such as
CPU/memory utilization or custom/external metrics. CPU utilization
commonly depends on CPU resource requests being set. The cluster
normally needs `metrics-server` for basic resource metrics.

Example:

``` yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spring-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spring-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Meaning: aim for average CPU usage around 70% of requested CPU, scaling
between 2 and 10 replicas. This is a target, not an instant guarantee.
HPA decisions have timing and stabilization behavior.

Check:

``` bash
kubectl get hpa
kubectl describe hpa spring-api-hpa
kubectl top pods
```

### VPA --- Vertical Pod Autoscaler

VPA recommends or adjusts CPU/memory requests (and in some modes limits)
for containers. Installation and behavior depend on the VPA
implementation and configuration. Applying new resource values may
restart Pods depending on mode and version. Avoid blindly using HPA and
VPA to control the same CPU/memory signal for the same workload, because
they can work against each other.

Illustrative VPA YAML (requires VPA components/CRDs):

``` yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: spring-api-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spring-api
  updatePolicy:
    updateMode: "Off"
```

`Off` is a recommendation-only style in common VPA implementations;
confirm the exact schema and behavior of the VPA version installed.

### CA --- Cluster Autoscaler

Cluster Autoscaler changes the number of worker nodes when Pods cannot
be scheduled due to insufficient capacity or nodes are underutilized and
safe to remove. It must be integrated with the infrastructure
provider/node groups and permissions. In EKS, use a supported node
autoscaling approach such as Cluster Autoscaler or Karpenter according
to your platform design; EKS Auto Mode has its own managed capabilities.

### How they work together

-   HPA: more/less Pods.
-   VPA: suitable CPU/memory per Pod.
-   Cluster Autoscaler: more/less worker nodes when supported and
    needed.
-   If HPA adds Pods but no node has capacity, a node autoscaler may add
    nodes---after correct setup. Scaling takes time and costs money.

------------------------------------------------------------------------

## 18. Ingress

### What is Ingress?

An Ingress is a Kubernetes API resource describing HTTP/HTTPS routing
from outside the cluster to Services, often based on hostnames and
paths.

**Important:** An Ingress resource by itself does not handle traffic.
You need an Ingress controller (for example, NGINX Ingress Controller or
an AWS Load Balancer Controller integration) that watches Ingress
objects and implements the routing.

### Ingress YAML example

This uses the common `networking.k8s.io/v1` API. The controller-specific
annotations and TLS setup vary:

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: spring-api
                port:
                  number: 80
```

Replace `app.example.com` and `nginx` with your actual domain and
installed controller class. The Service `spring-api` must exist in the
same namespace as the Ingress and point to healthy Pods.

### Ingress vs Service

-   Service gives stable access to a group of Pods.
-   Ingress defines HTTP/HTTPS routing rules to Services.
-   A `LoadBalancer` Service can expose a service directly.
-   Ingress typically consolidates multiple HTTP routes behind a
    controller/load balancer, depending on the controller.

For TLS, configure the controller and a TLS Secret or provider-specific
certificate integration. Do not assume the sample above enables HTTPS.

------------------------------------------------------------------------

## 19. How everything connects in a real application

Imagine an online shopping system with a frontend, Spring Boot API, and
database.

1.  A **Namespace** separates the team's application resources.
2.  A **Deployment** runs multiple stateless API Pods.
3.  The Deployment manages a **ReplicaSet**, which maintains the Pod
    count.
4.  Pod **labels** identify which app/version a Pod belongs to.
5.  A **Service** selects healthy API Pods and gives them a stable DNS
    name.
6.  Frontend Pods call the API Service; they do not hard-code individual
    Pod IPs.
7.  A **ConfigMap** supplies ordinary configuration; a **Secret**
    supplies sensitive values with access restricted.
8.  A **PVC** requests persistent storage for a workload that needs it.
9.  A **StatefulSet** is appropriate when replicas need stable
    identities and per-Pod storage.
10. A **DaemonSet** runs node-level agents on eligible worker nodes.
11. **HPA** adjusts API Pod replica count based on metrics.
12. **Cluster Autoscaler** or another node-scaling solution may add
    capacity if Pods cannot fit.
13. **Ingress** routes external HTTP traffic to the frontend/API Service
    when a controller is installed.
14. An **Operator** may manage complex systems such as databases if an
    appropriate operator is installed.
15. On AWS, **EKS** manages the Kubernetes control plane and integrates
    with AWS infrastructure and services.

### Simplified traffic flow

`User -> DNS -> Load Balancer / Ingress Controller -> Service -> Ready Pod -> Application`

The exact path depends on whether you use Ingress, Gateway API, a
LoadBalancer Service, a service mesh, or another design.

### Simplified management flow

`kubectl apply -> API server -> etcd -> controllers -> scheduler -> kubelet/runtime -> Pod`

------------------------------------------------------------------------

## 20. Useful kubectl commands

### Inspect

``` bash
kubectl get nodes
kubectl get namespaces
kubectl get pods -A
kubectl get deploy,rs,sts,ds,svc,ingress
kubectl get pvc,pv
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Logs and debugging

``` bash
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
kubectl exec -it <pod-name> -- sh
kubectl get endpointslices
kubectl get hpa
kubectl top pods
kubectl top nodes
```

### Deployments

``` bash
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment/<name> --replicas=3
```

### Validate YAML before applying

``` bash
kubectl apply --dry-run=client -f app.yaml
kubectl diff -f app.yaml
kubectl apply -f app.yaml
```

A client-side dry run checks the manifest locally; server-side
validation can catch additional API/schema/admission issues when using
`--dry-run=server`.

------------------------------------------------------------------------

## 21. Troubleshooting guide

### Pod stuck in `Pending`

Check `kubectl describe pod`. Common causes: insufficient CPU/memory,
unbound PVC, taints without tolerations, node selectors/affinity, quota,
or scheduling constraints.

### `ImagePullBackOff`

Check image name/tag, registry credentials, network connectivity, and
whether the node can access the image registry.

### `CrashLoopBackOff`

Inspect logs and previous logs; check startup command, environment,
Secret/ConfigMap references, health checks, memory limits, and
application dependencies.

### Service has no traffic

Confirm the Service selector matches Pod labels, Pods are Ready,
`targetPort` matches the actual listening port, and EndpointSlices
contain endpoints. Check NetworkPolicies and application
binding/address.

### PVC is `Pending`

Check StorageClass, CSI driver/provisioner, access mode, capacity,
topology, and cluster events.

### HPA has no metrics

Check that metrics-server or the configured metrics adapter is installed
and healthy, resource requests are set for utilization-based scaling,
and the HPA target exists.

### Ingress does not work

Confirm a controller exists and is healthy, `ingressClassName` matches,
DNS points to the correct address, backend Service/port are correct, and
controller logs/events show no errors.

------------------------------------------------------------------------

## 22. Production checklist

-   Use a Deployment for stateless APIs and configure multiple replicas
    where needed.
-   Set resource requests and sensible limits.
-   Configure readiness and liveness probes appropriately; add startup
    probes for slow-starting apps.
-   Use versioned image tags or immutable digests; avoid `latest` for
    controlled production releases.
-   Use rolling updates only when the app and data migrations are
    compatible with old and new versions coexisting.
-   Use Secrets securely; restrict RBAC, enable encryption at rest, and
    consider a managed secret store.
-   Use NetworkPolicies where supported and define least-privilege
    network access.
-   Set up metrics, logs, alerts, and resource quotas.
-   Back up persistent data and test restore procedures.
-   Test rollout and rollback procedures.
-   Plan node capacity, Pod disruption budgets, and topology
    spread/anti-affinity where appropriate.
-   Treat cluster upgrades, add-on compatibility, IAM, cloud networking,
    and storage drivers as planned operational work.

------------------------------------------------------------------------

## 23. Interview quick revision

**Q: What is Kubernetes?**\
Kubernetes is an open-source container orchestration platform used to
deploy, scale, and manage containerized applications.

**Q: What is a Pod?**\
A Pod is the smallest deployable unit in Kubernetes and can contain one
or more tightly coupled containers that share networking and may share
storage.

**Q: What is the difference between Deployment and ReplicaSet?**\
A ReplicaSet maintains the desired number of matching Pods. A Deployment
manages ReplicaSets and provides declarative updates and
rollout/rollback functionality.

**Q: What is the difference between Deployment and StatefulSet?**\
Deployment is normally used for interchangeable stateless replicas.
StatefulSet provides stable Pod identity and can provide per-Pod
persistent storage.

**Q: What is a Service?**\
A Service provides a stable network endpoint and DNS name for a set of
Pods selected by labels.

**Q: What is Ingress?**\
Ingress defines HTTP/HTTPS routing rules to Services; an Ingress
controller must implement those rules.

**Q: ConfigMap vs Secret?**\
ConfigMap is for non-sensitive configuration. Secret is for sensitive
data and requires appropriate access and storage security; base64 is not
encryption.

**Q: PV vs PVC?**\
PV represents a storage resource; PVC is a request for storage by a
workload or user.

**Q: HPA vs VPA vs Cluster Autoscaler?**\
HPA changes Pod replica count, VPA adjusts/recommends per-Pod resources,
and Cluster Autoscaler changes the number of worker nodes.

**Q: What is an Operator?**\
An Operator uses a CRD and controller to automate application-specific
operational tasks.

**Q: What is EKS?**\
Amazon EKS is AWS's managed Kubernetes service; AWS manages the control
plane while the workload/data-plane responsibilities depend on the
selected EKS setup.

**Q: How do Pods communicate?**\
Containers in the same Pod can communicate through localhost. Different
Pods communicate over the cluster network, usually using a Service DNS
name rather than unstable Pod IPs.

------------------------------------------------------------------------

## 24. Official references

Use official documentation when a field, API version, controller, cloud
integration, or feature depends on your cluster version.

-   Kubernetes concepts: https://kubernetes.io/docs/concepts/
-   Cluster architecture:
    https://kubernetes.io/docs/concepts/architecture/
-   Pods: https://kubernetes.io/docs/concepts/workloads/pods/
-   Workloads: https://kubernetes.io/docs/concepts/workloads/
-   Services and networking:
    https://kubernetes.io/docs/concepts/services-networking/
-   ConfigMaps:
    https://kubernetes.io/docs/concepts/configuration/configmap/
-   Secrets: https://kubernetes.io/docs/concepts/configuration/secret/
-   Storage: https://kubernetes.io/docs/concepts/storage/
-   Horizontal Pod Autoscaling:
    https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/
-   Ingress:
    https://kubernetes.io/docs/concepts/services-networking/ingress/
-   Amazon EKS architecture:
    https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html
-   Amazon EKS user guide:
    https://docs.aws.amazon.com/eks/latest/userguide/

------------------------------------------------------------------------

## Final learning advice

Do not try to memorize every YAML field in one sitting. Learn the
relationship between the resources first:

**Pod -\> ReplicaSet -\> Deployment -\> Service -\> Ingress**, then add
**Namespace, ConfigMap/Secret, storage, StatefulSet, DaemonSet,
autoscaling, Operators, and EKS**.

Build the examples in a local cluster (such as kind or minikube) before
using a paid cloud cluster. Check `kubectl describe`, events, logs,
labels, selectors, and endpoints whenever a resource does not behave as
expected.

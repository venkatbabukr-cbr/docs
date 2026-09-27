# ☸️ Kubernetes Architecture & Reference Cheat Sheet

This comprehensive cheat sheet maps core Kubernetes architectural concepts to their practical `kubectl` inspection commands and [K9s TUI](https://github.com/derailed/k9s) shortcuts.

---

## 📊 Big Picture Architecture Overview

| If you need to manage... | Use these concepts: | Primary Role |
| :--- | :--- | :--- |
| **Compute & Scaling** | Nodes → Deployments → Pods | Running application containers |
| **Networking & Routing** | Ingress → Services (ClusterIP / NodePort) | Directing traffic to applications |
| **Settings & Passwords** | ConfigMaps & Secrets | Injecting environment configuration |
| **Permanent Disks** | PersistentVolumeClaims (PVC) → PersistentVolumes (PV) | Preserving data across restarts |
| **Specialized Workloads** | StatefulSets, DaemonSets, Jobs / CronJobs | Handling databases, system daemons, and tasks |
| **Cluster Management & Security** | Namespaces & RBAC (Roles, ClusterRoles) | Security, isolation, and access control |

---

## 📝 Core Concepts & Inspection Commands

### 🚀 Compute & Scaling
* **Node:** The physical or virtual machine server providing the underlying CPU and memory infrastructure.
* **Pod:** The smallest deployable unit representing a single running application instance containing one or more containers.
* **Deployment:** An abstraction managing stateless pods, automating scaling, self-healing, and rolling updates.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get nodes`** | `no` | **`:node`** | Lists physical or virtual worker machines. |
| **`kubectl get pods`** | `po` | **`:pod`** or **`:pods`** | Lists all running application containers. |
| **`kubectl get deployments`** | `deploy` | **`:deploy`** or **`:dp`** | Shows managed application states and replica counts. |
| **`kubectl get replicasets`** | `rs` | **`:rs`** | Displays direct pod controllers managed by deployments. |

### 🌍 Cluster Management & Security (Including RBAC)
* **Namespace:** A virtual partitioning layer inside a physical Kubernetes cluster used to isolate environments and avoid naming conflicts.
* **RBAC (Role-Based Access Control):** The security framework that governs user and application permissions using Roles (allowed actions inside a namespace), ClusterRoles (cluster-wide actions), and their corresponding Bindings.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get namespaces`** | `ns` | **`:ns`** or **`:namespace`** | Displays all logical isolation boundaries. |
| **`kubectl get roles`** | *N/A* | **`:role`** | Lists permissions granted within a specific namespace. |
| **`kubectl get rolebindings`** | *N/A* | **`:rolebinding`** | Maps specific users/service accounts to Namespaced Roles. |
| **`kubectl get clusterroles`** | *N/A* | **`:clusterrole`** | Lists global, cluster-wide permissions definitions. |
| **`kubectl get clusterrolebindings`** | *N/A* | **`:clusterrolebinding`** | Maps specific users/service accounts to global ClusterRoles. |
| **`kubectl get serviceaccounts`** | `sa` | **`:sa`** | Lists identities created for inside-the-pod application code. |
| **`kubectl get componentstatuses`** | `cs` | **`:cs`** | Checks the health of core control plane elements. |
| **`kubectl get api-resources`** | *N/A* | **`:api-resources`** | Displays every resource type your cluster supports. |

### 🌐 Networking & Connectivity
* **ClusterIP:** The default service type. It creates a stable, internal-only virtual IP address to balance traffic across a group of ephemeral pods.
* **NodePort:** An extension of ClusterIP that opens a specific static port (30000–32767) on every node host, exposing the service directly to external networks.
* **Ingress:** An API object that manages external HTTP/HTTPS traffic entering the cluster, functioning as a smart reverse proxy providing URL-based routing and SSL/TLS termination.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get services`** | `svc` | **`:svc`** | Lists internal endpoints and external load balancers. |
| **`kubectl get ingresses`** | `ing` | **`:ing`** | Displays external routing rules and SSL/TLS paths. |
| **`kubectl get networkpolicies`** | `netpol` | **`:netpol`** | Shows internal traffic firewall rules between pods. |

### ⚙️ Settings & Passwords
* **ConfigMap:** A mechanism to inject plain-text configuration data, environment variables, or configuration files into your application containers.
* **Secret:** Similar to a ConfigMap but designed explicitly for sensitive credentials. It hides plain text using base64 encoding.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get configmaps`** | `cm` | **`:cm`** | Lists plain-text key-value application settings. |
| **`kubectl get secrets`** | *N/A* | **`:secret`** | Shows sensitive data configurations like passwords. |

### 💾 Permanent Disks
* **PersistentVolume (PV):** An actual storage resource in the cluster managed by cluster administrators.
* **PersistentVolumeClaim (PVC):** A storage request or "voucher" submitted by an application pod to dynamically request and lock down a specific amount of PV space.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get persistentvolumes`** | `pv` | **`:pv`** | Displays cluster-wide, provisioned storage disks. |
| **`kubectl get persistentvolumeclaims`** | `pvc` | **`:pvc`** | Lists specific storage requests made by pods. |
| **`kubectl get storageclasses`** | `sc` | **`:sc`** | Shows the storage profile rule providers available. |

### 🏗️ Specialized Workloads
* **StatefulSet:** Designed for stateful systems like databases. It guarantees that pods maintain a persistent, unique network identity and bind to the exact same disk storage even after being rescheduled.
* **DaemonSet:** Ensures that a single instance of a specific pod runs continuously across every single worker node (commonly used for `kube-proxy`, logging, and monitoring agents).
* **Job / CronJob:** Workloads designed to execute a task and terminate upon successful completion. Jobs run once immediately, while CronJobs execute on a recurring time schedule.

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get daemonsets`** | `ds` | **`:ds`** | Shows the background processes tracking node count. |
| **`kubectl get statefulsets`** | `sts` | **`:sts`** | Shows stateful applications like active databases. |
| **`kubectl get jobs`** | *N/A* | **`:job`** | Lists batch tasks designed to run until completion. |
| **`kubectl get cronjobs`** | `cj` | **`:cj`** | Lists scheduled, recurring automation tasks. |

---

## 🛠️ Powerful Command Modifiers vs. K9s Shortcuts

Instead of appending terminal flags, K9s relies on fast single-key interactive commands while viewing any list:

| `kubectl` Modifier | K9s Interactive Shortcut | Description |
| :--- | :--- | :--- |
| **`-A`** (All Namespaces) | Press **`0`** | Flips the current view to show resources from **all namespaces** simultaneously. |
| **`-n <namespace>`** | Press **`1`** to **`9`** *(or `:ns` to select)* | Filters the live view down to specific active namespaces. |
| **`-o wide`** | Press **`Ctrl + w`** | Toggles wide column view on/off (reveals Pod IPs, Node locations). |
| **`-o yaml`** | Press **`y`** | Drops immediately into the fully formatted raw YAML configuration file. |
| **`-w`** (Watch live) | *Automatic* | K9s streams and refreshes resource states **constantly in real-time**. |

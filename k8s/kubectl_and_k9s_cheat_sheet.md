# ☸️ Kubernetes Architecture & Reference Cheat Sheet

This comprehensive cheat sheet maps core Kubernetes architectural concepts to their practical `kubectl` commands and [K9s TUI](https://github.com/derailed/k9s) shortcuts.

**How to use this sheet:**
* **Universal Commands** covers the verbs that work on *any* resource. Learn these first.
* Each **Core Concept** section has a **Get commands** table and a **Concept-specific actions** table. Actions are tagged by intent: 🔍 **Inspect**, 🐞 **Debug**, or ✏️ **Change**.
* **Troubleshooting Pod States** at the end maps common failure statuses to their first diagnostic command.

---

## 📊 Big Picture Architecture Overview

| If you need to manage... | Use these concepts: | Primary Role |
| :--- | :--- | :--- |
| **Compute & Scaling** | Nodes → Deployments → Pods | Running application containers |
| **Networking & Routing** | Ingress → Services (ClusterIP / NodePort) | Directing traffic to applications |
| **Settings & Passwords** | ConfigMaps & Secrets | Injecting environment configuration |
| **Permanent Disks** | PersistentVolumeClaims (PVC) → PersistentVolumes (PV) | Preserving data across restarts |
| **Specialized Workloads** | StatefulSets, DaemonSets, Jobs / CronJobs | Handling databases, system daemons, and tasks |
| **Cluster Management** | Namespaces & RBAC | Security, isolation, and access control |

---

## 🧰 Universal Commands (Work on Any Resource)

Replace `<type>` with any resource type or short name (`po`, `deploy`, `svc`, `cm`, `pvc`, and so on).

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🔍 Inspect | **`kubectl describe <type> <name>`** | Press **`d`** | Shows full details plus recent **Events**. The first stop for almost any problem. |
| 🔍 Inspect | **`kubectl get <type> <name> -o yaml`** | Press **`y`** | Displays the live YAML, including the `status` block. |
| 🔍 Inspect | **`kubectl get events --sort-by=.lastTimestamp`** | **`:events`** | Lists cluster events in time order. Add `-n <ns>` to narrow down. |
| 🔍 Inspect | **`kubectl explain <type>.spec`** | *None* | Built-in field documentation. Drill deeper, e.g. `kubectl explain deploy.spec.strategy`. |
| 🔍 Inspect | *None* | **`:xray <type>`** | Shows a resource tree, e.g. Deployment → ReplicaSet → Pods → Containers. |
| ✏️ Change | **`kubectl apply -f <file.yaml>`** | *None* | Creates or updates resources declaratively from a manifest. The preferred way to change things. |
| ✏️ Change | **`kubectl diff -f <file.yaml>`** | *None* | Previews what `apply` would change. Run it before every `apply` on shared clusters. |
| ✏️ Change | **`kubectl edit <type> <name>`** | Press **`e`** | Opens the live resource in your editor. Good for quick fixes, but the change is lost on the next `apply` from Git. |
| ✏️ Change | **`kubectl delete <type> <name>`** | Press **`Ctrl + d`** | Deletes a resource (K9s asks for confirmation). |

---

## 📝 Core Concepts & Commands

### 🚀 Compute & Scaling
* **Node:** The physical or virtual machine server providing the underlying CPU and memory infrastructure.
* **Pod:** The smallest deployable unit representing a single running application instance containing one or more containers.
* **Deployment:** An abstraction managing stateless pods, automating scaling, self-healing, and rolling updates.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get nodes`** | `no` | **`:node`** | Lists physical or virtual worker machines. |
| **`kubectl get pods`** | `po` | **`:pod`** or **`:pods`** | Lists all running application containers. |
| **`kubectl get deployments`** | `deploy` | **`:deploy`** or **`:dp`** | Shows managed application states and replica counts. |
| **`kubectl get replicasets`** | `rs` | **`:rs`** | Displays direct pod controllers managed by deployments. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🔍 Inspect | **`kubectl logs <pod>`** | Press **`l`** on a pod | Prints container logs. Add **`-f`** to stream live. |
| 🔍 Inspect | **`kubectl logs <pod> --previous`** | Press **`p`** inside the log view | Shows logs from the **last crashed** container. Essential for CrashLoopBackOff. |
| 🔍 Inspect | **`kubectl logs <pod> -c <container>`** | Press **`Enter`** on the pod, select the container, then **`l`** | Targets one container in a multi-container pod. |
| 🔍 Inspect | **`kubectl top pods`** and **`kubectl top nodes`** | CPU and MEM columns in **`:pod`** and **`:node`** | Shows live CPU and memory usage. Requires metrics-server. |
| 🔍 Inspect | **`kubectl rollout status deploy/<name>`** | *None* | Waits and reports whether a rollout finished successfully. |
| 🔍 Inspect | **`kubectl rollout history deploy/<name>`** | *None* | Lists previous revisions available for rollback. |
| 🐞 Debug | **`kubectl exec -it <pod> -- sh`** | Press **`s`** on a pod | Opens a shell inside a running container. |
| 🐞 Debug | **`kubectl port-forward pod/<pod> 8080:80`** | Press **`Shift + f`** | Tunnels a local port to the pod for direct testing. |
| 🐞 Debug | **`kubectl debug -it <pod> --image=busybox:1.36 --target=<container>`** | *None* | Attaches a temporary debug container, useful when the image has no shell. |
| ✏️ Change | **`kubectl scale deploy/<name> --replicas=3`** | Press **`s`** on a deployment | Changes the replica count. |
| ✏️ Change | **`kubectl set image deploy/<name> <container>=<image>:<tag>`** | *None* (use **`e`**) | Triggers a rolling update to a new image. |
| ✏️ Change | **`kubectl rollout restart deploy/<name>`** | Press **`r`** on a deployment | Recreates all pods gradually without changing the spec. |
| ✏️ Change | **`kubectl rollout undo deploy/<name>`** | *None* | Rolls back to the previous revision. Add `--to-revision=<n>` to pick a specific one. |

---

### 🔌 Networking & Connectivity
* **CNI (Container Network Interface):** The system network engine (e.g., Calico, Cilium, Flannel) that provisions individual IP addresses for pods and establishes the virtual network mesh across the nodes.
* **ClusterIP:** The default service type. It creates a stable, internal-only virtual IP address to balance Layer 4 traffic across a group of ephemeral backend pods.
* **NodePort:** An extension of ClusterIP that opens a specific static port (30000–32767) on every node host, exposing the service directly to external networks.
* **Ingress:** An API object that manages external HTTP/HTTPS traffic entering the cluster, functioning as a smart reverse proxy providing URL-based routing and SSL/TLS termination.
* **Endpoints / EndpointSlices:** Dynamic, low-level tracker objects generated by Kubernetes that maintain the exact, live list of healthy pod IP addresses currently attached to a Service.
* **CoreDNS:** The internal, centralized cluster DNS infrastructure that resolves service text names (e.g., `http://my-service`) into their corresponding virtual ClusterIP addresses.
* **NetworkPolicy:** Firewalls inside the cluster that securely restrict or permit Layer 3/4 traffic flow between specific pods based on label selectors.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get services`** | `svc` | **`:svc`** | Lists internal endpoints and external load balancers. |
| **`kubectl get ingresses`** | `ing` | **`:ing`** | Displays external routing rules and SSL/TLS paths. |
| **`kubectl get endpointslices`** | `es` | **`:endpointslice`** | Inspects active backend pod IP tracking tables. |
| **`kubectl get networkpolicies`** | `netpol` | **`:netpol`** | Shows internal traffic firewall rules between pods. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🐞 Debug | **`kubectl get endpointslices -l kubernetes.io/service-name=<svc>`** | **`:endpointslice`** | Checks whether a Service has any backend pods. Empty endpoints usually mean the Service selector doesn't match the pod labels. |
| 🐞 Debug | **`kubectl port-forward svc/<svc> 8080:80`** | Press **`Shift + f`** on a service | Tests a Service from your machine, bypassing Ingress. |
| 🐞 Debug | **`kubectl run tmp --rm -it --image=busybox:1.36 --restart=Never -- sh`** | *None* | Starts a throwaway pod for in-cluster tests. Inside it, run `nslookup <svc>` for DNS or `wget -qO- http://<svc>:<port>` for connectivity. |

---

### ⚙️ Settings & Passwords
* **ConfigMap:** A mechanism to inject plain-text configuration data, environment variables, or configuration files into your application containers.
* **Secret:** Similar to a ConfigMap but intended for sensitive credentials (API keys, passwords). Values are **base64-encoded, not encrypted**: anyone with read access can decode them instantly. Protect Secrets by restricting read access through RBAC and enabling encryption at rest on the cluster.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get configmaps`** | `cm` | **`:cm`** | Lists plain-text key-value application settings. |
| **`kubectl get secrets`** | *None* | **`:secret`** | Shows sensitive data configurations like passwords. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🔍 Inspect | **`kubectl get secret <name> -o jsonpath='{.data.<key>}' \| base64 -d`** | Press **`x`** on a Secret | Decodes a Secret value to plain text for verification. |
| ✏️ Change | **`kubectl create configmap <name> --from-file=<file>`** | *None* | Creates a ConfigMap from a file. Add `--dry-run=client -o yaml` to generate a manifest instead. |
| ✏️ Change | **`kubectl create secret generic <name> --from-literal=KEY=value`** | *None* | Creates a Secret. Literal values land in your shell history, so prefer `--from-file` for real credentials. |
| ✏️ Change | **`kubectl rollout restart deploy/<name>`** | Press **`r`** on the deployment | Required after changing a ConfigMap or Secret used as environment variables. Pods do not pick up the change on their own. |

---

### 💾 Permanent Disks & Storage
* **PersistentVolume (PV):** An actual storage resource in the cluster (like a cloud disk or network-attached storage) managed by cluster administrators.
* **PersistentVolumeClaim (PVC):** A storage request or "voucher" submitted by an application pod to dynamically request and lock down a specific amount of PV space.
* **StorageClass:** A dynamic storage profile provider that allows cloud volumes to be generated automatically on-demand when a PVC is created.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get persistentvolumes`** | `pv` | **`:pv`** | Displays cluster-wide, provisioned storage disks. |
| **`kubectl get persistentvolumeclaims`** | `pvc` | **`:pvc`** | Lists specific storage requests made by pods. |
| **`kubectl get storageclasses`** | `sc` | **`:sc`** | Shows the storage profile rule providers available. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🐞 Debug | **`kubectl describe pvc <name>`** | Press **`d`** on a PVC | Explains a PVC stuck in **Pending** through its Events: missing StorageClass, no capacity, or `WaitForFirstConsumer` (binds only once a pod uses it). |
| ✏️ Change | **`kubectl patch pvc <name> -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'`** | *None* (use **`e`**) | Expands a volume. Works only if the StorageClass has `allowVolumeExpansion: true`. Volumes cannot shrink. |

---

### 🏗️ Specialized Workloads
* **StatefulSet:** Designed for stateful systems like databases. It guarantees that pods maintain a persistent, unique network identity and bind to the exact same disk storage even after being rescheduled.
* **DaemonSet:** Ensures that a single instance of a specific pod runs continuously across every single worker node (commonly used for `kube-proxy`, logging, and monitoring agents).
* **Job / CronJob:** Workloads designed to execute a task and terminate upon successful completion. **Jobs** run once immediately (e.g., database migrations), while **CronJobs** execute on a recurring time schedule.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get daemonsets`** | `ds` | **`:ds`** | Shows background processes tracking node count. |
| **`kubectl get statefulsets`** | `sts` | **`:sts`** | Shows stateful applications like active databases. |
| **`kubectl get jobs`** | *None* | **`:job`** | Lists batch tasks designed to run until completion. |
| **`kubectl get cronjobs`** | `cj` | **`:cj`** | Lists scheduled, recurring automation tasks. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🔍 Inspect | **`kubectl logs job/<name>`** | Press **`l`** on a job | Shows the output of a Job's pod, including completed ones. |
| 🔍 Inspect | **`kubectl rollout status sts/<name>`** | *None* | Tracks a StatefulSet or DaemonSet rollout (use `ds/<name>` for DaemonSets). StatefulSet pods update one at a time, in order. |
| ✏️ Change | **`kubectl create job <name> --from=cronjob/<cronjob>`** | Press **`t`** on a CronJob | Triggers a CronJob immediately, without waiting for its schedule. |
| ✏️ Change | **`kubectl patch cronjob <name> -p '{"spec":{"suspend":true}}'`** | Press **`s`** on a CronJob | Pauses a CronJob. Set `suspend` to `false` to resume. |
| ✏️ Change | **`kubectl rollout restart sts/<name>`** | Press **`r`** on a StatefulSet | Restarts StatefulSet pods one at a time. Works for DaemonSets too (`ds/<name>`). |

---

### 🛡️ Cluster Management, Isolation & Security (RBAC)
* **Namespace:** A virtual partitioning layer inside a physical Kubernetes cluster used to isolate environments (like `development`, `staging`, and `production`) and avoid resource naming conflicts.
* **RBAC (Role-Based Access Control):** The security framework that governs user and application permissions using **Roles** (defining allowed actions) and **RoleBindings** (assigning those actions to specific users or service accounts).
* **ServiceAccount:** An identity created explicitly for **inside-the-pod application code** to securely authenticate against the cluster's control plane API.

#### Get commands

| `kubectl` Command | Short Name | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get namespaces`** | `ns` | **`:ns`** or **`:namespace`** | Displays all logical isolation boundaries. |
| **`kubectl get roles`** | *None* | **`:role`** | Lists local permissions allowed inside a **single specific namespace**. |
| **`kubectl get rolebindings`** | *None* | **`:rolebinding`** | Maps a local namespace `Role` to a user or pod group. |
| **`kubectl get clusterroles`** | *None* | **`:clusterrole`** | Lists global permissions that apply **across the entire cluster**. |
| **`kubectl get clusterrolebindings`** | *None* | **`:clusterrolebinding`** | Maps a global `ClusterRole` to a user or pod group globally. |
| **`kubectl get serviceaccounts`** | `sa` | **`:sa`** | Lists programmatic identities assigned to pods. |
| **`kubectl get --raw='/readyz?verbose'`** | *None* | **`:pulse`** | Checks control plane health check by check (`componentstatuses` is deprecated since v1.19). |
| **`kubectl get api-resources`** | *None* | **`:api-resources`** | Displays every resource type your cluster supports. |

#### Concept-specific actions

| Intent | `kubectl` Command | K9s Shortcut | Description |
| :--- | :--- | :--- | :--- |
| 🔍 Inspect | **`kubectl auth can-i <verb> <resource> -n <ns>`** | *None* | Checks whether *you* can perform an action, e.g. `kubectl auth can-i delete pods -n staging`. |
| 🔍 Inspect | **`kubectl auth can-i --list -n <ns>`** | *None* | Lists everything you are allowed to do in a namespace. |
| 🐞 Debug | **`kubectl auth can-i <verb> <resource> -n <ns> --as=system:serviceaccount:<ns>:<sa>`** | *None* | Tests a ServiceAccount's permissions. The fastest way to debug `403 Forbidden` errors from pods. |
| ✏️ Change | **`kubectl create rolebinding <name> --role=<role> --serviceaccount=<ns>:<sa> -n <ns>`** | *None* | Grants a Role to a ServiceAccount. Use `--user=<name>` for a human user. |

---

## 🩺 Troubleshooting Pod States

| Pod Status | Likely Cause | First Command to Run |
| :--- | :--- | :--- |
| **Pending** | Not enough CPU or memory on any node, an unbound PVC, or node taints the pod doesn't tolerate. | `kubectl describe pod <pod>` and read **Events** |
| **ImagePullBackOff** or **ErrImagePull** | Wrong image name or tag, missing `imagePullSecrets` for a private registry, or the registry is unreachable. | `kubectl describe pod <pod>` |
| **CrashLoopBackOff** | The app exits on startup: bad configuration, a missing dependency, or a failing liveness probe. | `kubectl logs <pod> --previous` |
| **CreateContainerConfigError** | A referenced ConfigMap, Secret, or key inside one doesn't exist. | `kubectl describe pod <pod>` |
| **OOMKilled** | The container exceeded its memory limit. | `kubectl describe pod <pod>` (check **Last State**), then `kubectl top pod <pod>` |
| **Evicted** | The node ran low on memory or disk and removed pods. | `kubectl describe node <node>` (check **Conditions**) |
| **Running but not Ready** | The readiness probe is failing, so the pod receives no traffic. | `kubectl describe pod <pod>` and read **Events** |
| **Service unreachable** (pods healthy) | The Service selector doesn't match the pod labels, so it has no endpoints. | `kubectl get endpointslices -l kubernetes.io/service-name=<svc>` |

---

## 🛠️ Powerful Command Modifiers vs. K9s Shortcuts

Instead of appending terminal flags, K9s relies on fast single-key interactive commands while viewing any list:

| `kubectl` Modifier | K9s Interactive Shortcut | Description |
| :--- | :--- | :--- |
| **`-A`** (All Namespaces) | Press **`0`** | Flips the current view to show resources from **all namespaces** simultaneously. |
| **`-n <namespace>`** | Press **`1`** to **`9`** *(or `:ns` and select)* | Switches to one of your favorite namespaces shown in the K9s header. Use `:ns` to pick any other namespace, which then gets added to the favorites. |
| **`-o wide`** | Press **`Ctrl + w`** | Toggles wide column view on and off (reveals Pod IPs, Node locations). |
| **`-o yaml`** | Press **`y`** | Drops immediately into the fully formatted raw YAML configuration file. |
| **`-w`** (Watch live) | *Automatic* | K9s streams and refreshes resource states **constantly in real-time**. |

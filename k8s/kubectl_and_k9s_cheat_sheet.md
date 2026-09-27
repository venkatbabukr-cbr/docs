# All gets

`kubectl get` commands and their corresponding [K9s TUI](https://github.com/derailed/k9s) shortcuts.

---

## 🌍 Cluster-Level & Node Infrastructure

| `kubectl` Command | Short Name / Alias | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get nodes`** | `no` | **`:node`** | Lists all physical or virtual worker machines in the cluster. |
| **`kubectl get namespaces`** | `ns` | **`:ns`** or **`:namespace`** | Displays all logical isolation boundaries. |
| **`kubectl get componentstatuses`** | `cs` | **`:cs`** | Checks the health of core control plane elements. |
| **`kubectl get api-resources`** | *N/A* | **`:api-resources`** | Displays every resource type your cluster supports. |

## 🚀 Workloads & Applications

| `kubectl` Command | Short Name / Alias | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get pods`** | `po` | **`:pod`** or **`:pods`** | Lists all running application containers. |
| **`kubectl get deployments`** | `deploy` | **`:deploy`** or **`:dp`** | Shows managed application states and replica counts. |
| **`kubectl get daemonsets`** | `ds` | **`:ds`** | Shows the background processes tracking node count. |
| **`kubectl get statefulsets`** | `sts` | **`:sts`** | Shows stateful applications like active databases. |
| **`kubectl get replicasets`** | `rs` | **`:rs`** | Displays the direct pod controllers managed by deployments. |
| **`kubectl get jobs`** | *N/A* | **`:job`** | Lists batch tasks designed to run until completion. |
| **`kubectl get cronjobs`** | `cj` | **`:cj`** | Lists scheduled, recurring automation tasks. |

## 🔌 Networking & Connectivity

| `kubectl` Command | Short Name / Alias | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get services`** | `svc` | **`:svc`** | Lists internal endpoints and external load balancers. |
| **`kubectl get ingresses`** | `ing` | **`:ing`** | Displays external routing rules and SSL/TLS paths. |
| **`kubectl get networkpolicies`** | `netpol` | **`:netpol`** | Shows internal traffic firewall rules between pods. |

## 💾 Configuration & Storage

| `kubectl` Command | Short Name / Alias | K9s Command Mode | Description |
| :--- | :--- | :--- | :--- |
| **`kubectl get configmaps`** | `cm` | **`:cm`** | Lists plain-text key-value application settings. |
| **`kubectl get secrets`** | *N/A* | **`:secret`** | Shows sensitive data configurations like passwords. |
| **`kubectl get persistentvolumes`** | `pv` | **`:pv`** | Displays cluster-wide, provisioned storage disks. |
| **`kubectl get persistentvolumeclaims`** | `pvc` | **`:pvc`** | Lists specific storage requests made by pods. |
| **`kubectl get storageclasses`** | `sc` | **`:sc`** | Shows the storage profile rule providers available. |

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
# 🐶 K9s Quick Guide

K9s is a terminal UI that wraps `kubectl`. Every K9s shortcut in this sheet follows a small set of rules, so learning the rules first makes the individual keys easy to recall.

## K9s Key Bindings Logic

K9s bindings work in seven layers, each with its own rule.

### 1. Vim handles navigation and modes

These come straight from Vim and stay consistent across every view:

* **`:`** enters command mode, like Vim's ex commands. Type a resource name to jump to it.
* **`/`** filters the current list, like Vim's search. It also understands:
  * `/api|web`: regex, which is the default.
  * `/!Running`: inverse, showing everything *not* Running. Handy for spotting broken pods.
  * `/-l app=web`: a label selector, like `kubectl get -l`.
  * `/-f ngx`: a fuzzy match.
* **`j`** and **`k`** move down and up. **`Esc`** backs out of whatever you're in.
* **`?`** shows help, as in Vim and `less`.

### 2. Command mode reuses kubectl's short names

Whatever you'd type after `kubectl get`, you type after `:`. `kubectl get deploy` becomes `:deploy`, and `kubectl get pvc` becomes `:pvc`. Knowing kubectl short names means you already know K9s navigation. When unsure, type `:` plus a partial name to autocomplete.

### 3. Plain letters are safe verbs on the selected item

A single lowercase key is the first letter of the action, and it never destroys anything:

| Key | Mnemonic |
| :--- | :--- |
| `d` | **d**escribe |
| `y` | **y**aml |
| `e` | **e**dit |
| `l` | **l**ogs |
| `s` | **s**hell (on a pod), **s**cale (on a Deployment), **s**uspend (on a CronJob) |
| `p` | **p**revious logs (inside the log view) |
| `r` | **r**estart |
| `t` | **t**rigger (CronJobs) |

Describe gets the easy single key because it's harmless, which is exactly why delete does *not*.

### 4. Ctrl means "careful" or "change the view itself"

Ctrl is the guard rail. It marks either something destructive or something that affects the whole screen rather than one item:

* **Destructive:** `Ctrl + d` **d**elete (with confirmation), `Ctrl + k` **k**ill (force-delete, no confirmation).
* **View-level:** `Ctrl + w` toggles **w**ide columns, `Ctrl + a` lists **a**ll aliases.

The rule: a plain letter acts on the item, a Ctrl letter is dangerous or global.

### 5. Shift sorts, numbers pick namespaces

* **Shift + letter** sorts by the column starting with that letter: `Shift + c` **C**PU, `Shift + m` **M**emory, `Shift + n` **N**ame, `Shift + a` **A**ge. The notable exception is `Shift + f` for port-**f**orward.
* **`0`** shows all namespaces, and **`1`** to **`9`** switch to your favorites. Think of `0` as "no filter."

### 6. Log view keys (after pressing `l`)

Inside the log view, plain letters toggle how the stream is shown rather than acting on a resource:

| Key | Action |
| :--- | :--- |
| `s` | Toggle auto-scroll, to pause a fast stream |
| `w` | Toggle line wrap |
| `t` | Toggle timestamps |
| `f` | Toggle full screen |
| `/` | Filter log lines |
| `Ctrl + s` | Save the logs to a file |

### 7. Bulk actions

Actions normally apply to the row under the cursor. Mark rows first, and the next action applies to all of them:

| Key | Action |
| :--- | :--- |
| `Space` | Mark or unmark the current row |
| `Ctrl + Space` | Mark every row between the last mark and the cursor |
| `Ctrl + \` | Clear all marks |

Then press the action key, for example `Ctrl + d`, to apply it to every marked row.

### 8. Two things that trip people up

* **The same letter changes meaning by view.** The letter maps to *the most natural action for that resource*: pods have no replicas to scale, and Deployments have no single container to shell into, so there's no collision in practice. The hint bar at the top of every view lists the keys valid there, so glance up rather than memorizing per-view tables.
* **`Enter` and `Esc` follow ownership.** `Enter` drills down the hierarchy (Deployment → Pods → Containers), and `Esc` climbs back up. It mirrors the resource tree that `:xray` shows.

## K9s-Only Views (No kubectl Equivalent)

These views go beyond wrapping `kubectl`. Open them from command mode like any resource.

| Command | What it shows | When to reach for it |
| :--- | :--- | :--- |
| **`:pulse`** (`:pu`) | A live dashboard of the cluster: counts and health of Pods, Deployments, StatefulSets, DaemonSets, Jobs, PVs, Events, plus node CPU and memory. | First screen when you open an unfamiliar cluster, or a "is anything on fire?" glance. Press `Enter` on a tile to jump to that resource. |
| **`:xray <type>`** | The ownership tree for a resource type, e.g. `:xray deploy` shows Deployment → ReplicaSet → Pods → Containers, with ConfigMaps, Secrets and ServiceAccounts each pod uses. | Answering "what does this Deployment actually run and depend on?" Red items mark unhealthy children. |
| **`:pf`** | All active port-forwards started from K9s (`Shift + f`). | Seeing and stopping forwards you've opened, so they don't linger. |
| **`:benchmarks`** (`:be`) | Results of HTTP load tests. Start one with `Ctrl + l` on a row in `:pf`. | Quick latency and throughput check of a service without installing a load tool. |
| **`:rbac`** / **`:policy`** | Effective permissions for a Role or ClusterRole, or for a user, group or ServiceAccount. | Debugging `Forbidden` errors. Easier to read than `kubectl auth can-i --list`. |
| **`:dir <path>`** | A file browser for local manifests. Press `a` to apply the selected file. | Applying YAML without leaving K9s. |
| **`:ctx`** / **`:ns`** | Contexts and namespaces. Press `Enter` to switch, `u` to mark a namespace as a favorite. | Moving between clusters, and filling the `1` to `9` favorites. |
| **`:screendump`** (`:sd`) | Files saved with `Ctrl + s` from any table view (as CSV) or from logs. | Finding the snapshot you just saved to share with a teammate. |

## K9s Startup & Customization

**Useful startup flags:**

* `k9s -n <ns> -c deploy` opens straight into a namespace and view.
* `k9s --context <ctx>` picks the cluster without switching your kubeconfig.
* `k9s --readonly` disables every change and delete action. Use it on production.
* `k9s info` prints where K9s keeps its config, logs and screen dumps.

**Customization files** (in the config directory that `k9s info` prints):

* `aliases.yaml` adds your own command-mode shortcuts, e.g. `dp: deployments`.
* `hotkeys.yaml` binds keys like `Shift + 0` to a view, e.g. jump straight to `:pulse`.
* `plugins.yaml` adds custom keys that run shell commands on the selected item, e.g. run `stern` for multi-pod logs or `kubectl debug` on a pod.
* Skins (`skins/*.yaml`) change the color theme.

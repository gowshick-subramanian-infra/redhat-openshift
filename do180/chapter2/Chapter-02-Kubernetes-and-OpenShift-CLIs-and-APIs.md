# Chapter 2 — Kubernetes and OpenShift Command-Line Interfaces and APIs

> DO180 study notes

## 2.1 The Kubernetes and OpenShift Command-Line Interfaces

### Command-line interfaces

A command-line interface (CLI) is used to interact with a cluster by executing commands from a terminal.

| CLI | Used with | Description |
|---|---|---|
| `oc` | Red Hat OpenShift | OpenShift CLI. Supports Kubernetes operations and OpenShift-specific functionality. |
| `kubectl` | Kubernetes | Standard Kubernetes CLI used to manage Kubernetes resources. |

OpenShift is based on Kubernetes, so most common Kubernetes operations can be performed using either `oc` or `kubectl`.

Examples:

```bash
oc get nodes
oc get pods
oc get services
oc get pv
oc get sc
oc describe pod <pod-name>
oc edit deployment <deployment-name>
```

Equivalent Kubernetes commands:

```bash
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get pv
kubectl get sc
kubectl describe pod <pod-name>
kubectl edit deployment <deployment-name>
```

### `oc` versus `kubectl`

- `kubectl` supports standard Kubernetes resource operations.
- `oc` supports those Kubernetes operations and additional OpenShift-specific commands.
- OpenShift-specific commands such as `oc new-project` are not available in `kubectl`.
- OpenShift resources and features may require `oc` commands or OpenShift-specific API resources.

Examples of OpenShift-specific commands:

```bash
oc new-project <project-name>
oc get imagestreams
oc extract secret/<secret-name>
```

Use `oc` as the primary CLI when administering an OpenShift cluster. `kubectl` remains useful when working with standard Kubernetes resources and tooling.

---

## 2.2 Guided Exercise — The Kubernetes and OpenShift Command-Line Interfaces

### Where to install the CLI

Install the OpenShift CLI on a machine that can reach the cluster API endpoint. This can be:
- An approved jump host.
- A Linux workstation.
- A Linux virtual machine running locally, for example in Oracle VirtualBox.

The workstation or VM needs network connectivity to the OpenShift API endpoint.

### Recommended download source

Use the Red Hat Hybrid Cloud Console:

[https://console.redhat.com/openshift/downloads](https://console.redhat.com/openshift/downloads)

General workflow:

1. Sign in to the Red Hat console.
2. Open the OpenShift downloads page.
3. Select the OpenShift command-line interface.
4. Choose the operating system and CPU architecture.
5. Select the required OpenShift client release.
6. Download the archive or copy its download link.
7. Download the archive to the Linux workstation.
8. Extract the archive and install the binaries.

### Download and extract on Linux

The following example uses a copied download URL. Replace the URL and filename with the values for the required release and platform.

```bash
mkdir -p ~/openshift-cli
cd ~/openshift-cli

wget '<download-url>'
tar -xvf <downloaded-archive>.tar.gz
```

The extracted archive normally contains the `oc` and `kubectl` client binaries.

Check the extracted files:

```bash
ls -l
```

Check the client versions from the current directory:

```bash
./oc version --client
./kubectl version --client
```

### Install the binaries in the system PATH

For a system-wide installation:

```bash
sudo install -m 0755 oc /usr/local/bin/oc
sudo install -m 0755 kubectl /usr/local/bin/kubectl
```

Verify:

```bash
oc version --client
kubectl version --client
which oc
which kubectl
```

If the binaries are stored in a directory that is not in `PATH`, the shell will not find them unless the full path is used.

### Add a personal binary directory to PATH

The CLI can also be kept in a user's home directory:

```bash
mkdir -p ~/bin
cp oc kubectl ~/bin/
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
oc version --client
kubectl version --client
```

### Select a compatible client version

The OpenShift CLI version should be compatible with the target cluster version.

Recommended approach:
- Check the OpenShift cluster version.
- Download the matching `oc` client release when possible.
- Avoid selecting an unrelated latest client release without checking compatibility.
- Keep the client version aligned with the cluster version used in the environment.

Check the cluster version after logging in:

```bash
oc get clusterversion
```

Check client and server versions:

```bash
oc version
```

The output distinguishes the client version from the server version. A version mismatch does not automatically mean the client will fail, but using a compatible client version reduces the risk of unexpected behavior.

### Download the CLI from the OpenShift web console

An alternative is to download the CLI directly from the OpenShift web console:

1. Sign in to the OpenShift web console.
2. Open the **Command Line Tools** option from the console help menu.
3. Select the appropriate operating system and architecture.
4. Download the CLI package for that cluster.
5. Extract and install the binaries.

This method is useful when a user does not have a separate Red Hat account or needs the CLI version provided for the cluster being accessed.

### Installation methods to avoid as the default

- Avoid downloading an arbitrary latest binary without checking the cluster version.
- Avoid relying on an unrelated or manually maintained package repository for the CLI when the supported Red Hat download is available.
- Avoid copying a client binary from an unknown source.

---

## 2.3 Inspect Kubernetes Resources

### Authenticate to the cluster

The CLI communicates with the cluster through the Kubernetes API server.

A typical OpenShift API endpoint uses this pattern:

```text
https://api.<cluster-domain>:6443
```

The API server commonly listens on TCP port `6443`.

Log in using a username when that authentication method is enabled:

```bash
oc login https://api.<cluster-domain>:6443 -u <username>
```

The CLI prompts for the password.

Alternatively, authenticate using a token:

```bash
oc login https://api.<cluster-domain>:6443 --token=<token>
```

Treat tokens as secrets. Do not publish them in scripts, screenshots, Git repositories or shared notes.

### Verify the login

```bash
oc whoami
oc whoami --show-server
oc cluster-info
```

- `oc whoami` displays the authenticated identity.
- `oc whoami --show-server` displays the API server URL.
- `oc cluster-info` displays cluster endpoint information.

### Authentication identity types

#### Regular users
- Human user identities used to access the cluster.
- May be backed by an external identity provider such as LDAP or Active Directory.
- May also be configured through another supported identity provider.

#### System users
- Identities used by cluster components and system processes.
- OpenShift includes system identities required for internal platform operations.
- These are different from ordinary human user accounts.

#### Service accounts
- Identities intended for processes and applications running in the cluster.
- A workload can use a service account when it needs to communicate with the Kubernetes API.
- Permissions should be granted according to the workload's requirements.

Service accounts and detailed authentication/authorization configuration are important OpenShift administration topics.

### Projects and namespaces

A project is the OpenShift-oriented way to organize resources. It is based on a Kubernetes namespace.

List projects:

```bash
oc get projects
```

List namespaces:

```bash
oc get namespaces
```

Display the current project:

```bash
oc project
```

Switch to a project:

```bash
oc project <project-name>
```

Create a project:

```bash
oc new-project <project-name>
```

---

## 2.4 Guided Exercise — Inspect Kubernetes Resources

### List resources

The general command structure is:

```bash
oc get <resource-type>
```

Examples:

```bash
oc get nodes
oc get pods
oc get deployments
oc get services
oc get routes
oc get pvc
oc get pv
oc get sc
```

Short names can also be used when supported:

```bash
oc get pod
oc get svc
oc get pvc
oc get pv
oc get sc
```

### List resources across namespaces

```bash
oc get pods --all-namespaces
```

Short form:

```bash
oc get pods -A
```

### Understand `oc get all`

```bash
oc get all
```

This displays a selected set of common application resources in the current namespace/project.

It does **not** display every resource in the namespace. ConfigMaps, Secrets, PersistentVolumeClaims and ServiceAccounts may need to be queried separately.

```bash
oc get configmaps
oc get secrets
oc get pvc
oc get serviceaccounts
```

### Describe a resource

Use `oc describe` to inspect detailed information about a resource.

```bash
oc describe node <node-name>
oc describe pod <pod-name>
oc describe deployment <deployment-name>
```

The output can include:
- Resource metadata.
- Labels and annotations.
- Current status and conditions.
- Pod scheduling information.
- Container details.
- Events associated with the resource.

`oc describe` is one of the first commands to use when investigating a resource that is not behaving as expected.

### Explain a resource field

Use `oc explain` to view documentation for Kubernetes resource types and their fields.

```bash
oc explain pod
oc explain deployment
oc explain node
```

To inspect a field:

```bash
oc explain pod.spec
oc explain pod.spec.containers
```

This is useful when writing or reviewing YAML manifests because it shows the structure and meaning of API fields.

### Inspect API resources and versions

```bash
oc api-resources
oc api-versions
```

- `oc api-resources` lists API resource types available in the cluster.
- `oc api-versions` lists API versions supported by the API server.

These commands help identify resource names and API groups available in the cluster.

### Create resources from a YAML file

Use `oc create -f` to create resources defined in a YAML manifest:

```bash
oc create -f <file-name>.yaml
```

For example:

```bash
oc create -f deployment.yaml
```

The manifest must contain a valid Kubernetes/OpenShift resource definition.

### Generate a YAML template without creating the resource

A client-side dry run can generate a YAML representation for review:

```bash
oc create deployment <deployment-name> \
 --image=<image-name> \
 --dry-run=client \
 -o yaml
```

To save the generated YAML:

```bash
oc create deployment <deployment-name> \
 --image=<image-name> \
 --dry-run=client \
 -o yaml > deployment.yaml
```

Review the generated manifest before applying it.

---

## 2.5 Assess the Health of an OpenShift Cluster

### Cluster information

```bash
oc cluster-info
oc get nodes
oc get clusterversion
oc get co
```

`oc get co` lists the OpenShift ClusterOperators.

### Cluster Operators

Cluster Operators manage and report the condition of major OpenShift platform capabilities.

Examples include operators responsible for:
- Authentication.
- Console.
- API server.
- Ingress.
- Image registry.
- Storage.
- Networking.
- Machine configuration.
- Monitoring.

List the operators:

```bash
oc get clusteroperators
```

Short form:

```bash
oc get co
```

### Understand ClusterOperator conditions

| Condition | Expected healthy value | Meaning |
|---|---|---|
| `Available` | `True` | The operator is available and functioning. |
| `Progressing` | `False` | The operator is not currently performing an update or rollout. |
| `Degraded` | `False` | The operator is not reporting a degraded state. |

A typical healthy operator should generally show:
- `Available=True`
- `Progressing=False`
- `Degraded=False`

Interpret the conditions together. During an OpenShift update, `Progressing=True` can be expected. A degraded operator requires investigation.

Inspect one operator in detail:

```bash
oc describe clusteroperator <operator-name>
```

### Check the cluster version

```bash
oc get clusterversion
```

This displays the cluster version and update-related status.

### Check node status

```bash
oc get nodes
```

A node in `NotReady` state needs investigation.

```bash
oc describe node <node-name>
```

### Inspect node CPU and memory capacity

List the nodes:

```bash
oc get nodes
```

Inspect an individual node:

```bash
oc describe node <node-name>
```

Important fields in the output:

| Field | Meaning |
|---|---|
| `Capacity` | Total resource capacity reported by the node |
| `Allocatable` | Resources available for Pods after node/system reservations |
| `Allocated resources` | Summary of workload resource requests and limits |

Do not treat `Capacity` and `Allocatable` as the same value. Not all node capacity is available for application Pods.

View current node resource usage:

```bash
oc adm top nodes
```

View current Pod resource usage:

```bash
oc adm top pods
oc adm top pods -A
```

`oc adm top` reports current usage when the cluster metrics pipeline is available. It does not replace the node's capacity and allocatable values.

For a cluster-wide view, use the OpenShift web console's monitoring dashboards when available. Compare node capacity, allocatable resources and current usage rather than manually adding request/limit values as though they were physical capacity.

### Check application and Pod status

```bash
oc get pods
oc get pods -A
```

Look for statuses such as:
- `Pending`
- `ImagePullBackOff`
- `ErrImagePull`
- `CrashLoopBackOff`
- `Error`

Use `describe` and logs to investigate the reason.

### Common troubleshooting commands

A useful initial troubleshooting sequence is:

```bash
oc get <resource>
oc describe <resource> <name>
oc logs <pod-name>
oc get events --sort-by=.lastTimestamp
```

For a Pod with multiple containers:

```bash
oc logs <pod-name> -c <container-name>
```

For logs from the previous container instance:

```bash
oc logs <pod-name> --previous
```

The description, logs and events provide different information and should be reviewed together.

### Troubleshoot image pull failures

An image pull failure can prevent a Pod from starting.

Start with:

```bash
oc get pods
oc describe pod <pod-name>
```

Review the Events section for messages such as:
- Image not found.
- Incorrect registry or repository path.
- Invalid image tag.
- Registry authentication failure.

Check the image reference in the workload definition:

```bash
oc describe deployment <deployment-name>
```

The image reference should contain the correct registry, repository and tag.

If the image is public and Podman is available in a suitable environment, the image reference can also be tested manually:

```bash
podman pull <registry>/<repository>:<tag>
```

A successful manual pull does not guarantee that the cluster can pull the image: the cluster may use different network access or registry credentials.

---

## 2.6 Guided Exercise — Assess the Health of an OpenShift Cluster

### Cluster health checklist

```bash
oc whoami
oc whoami --show-server
oc get clusterversion
oc get co
oc get nodes
oc get pods -A
oc get events -A --sort-by=.lastTimestamp
```

Review the output and confirm:

- [ ] The CLI is connected to the expected API endpoint.
- [ ] The cluster version is identified.
- [ ] Cluster Operators are available.
- [ ] Operators are not unexpectedly progressing or degraded.
- [ ] Nodes are in the expected state.
- [ ] Workload Pods are running and ready.
- [ ] Recent events do not show unresolved errors.

---

### Node-Level Troubleshooting and Emergency Cluster Access

### Use `oc debug` for node investigation

When the API server is available and the node can be reached through the cluster, `oc debug` can be used to start a debugging session for a node.

```bash
oc debug node/<node-name>
```

This provides a troubleshooting environment associated with the selected node.

Use node debugging for investigation. Avoid making persistent manual changes to a node unless the procedure is supported and understood.

### When a node is NotReady

If a node is `NotReady`, the normal debug workflow may not be available. First inspect the node and its underlying infrastructure.

```bash
oc get nodes
oc describe node <node-name>
```

Check whether the VM or physical server is powered on and reachable through the infrastructure management platform.

If authorized SSH access is available, connect to the node and inspect the key services.

```bash
sudo systemctl status kubelet
sudo systemctl status crio
```

Review service logs:

```bash
sudo journalctl -u kubelet
sudo journalctl -u crio
```

The kubelet registers and manages the node's workloads. CRI-O provides the container runtime used by OpenShift. If either service is unhealthy, the node may fail to become Ready.

Do not restart services blindly. Review the service status and logs first, identify the cause, and follow the approved recovery procedure.

### Check for node identity or network changes

A node may fail to register correctly if its IP address or hostname changes unexpectedly.

Check:
- Node IP address.
- Hostname.
- DNS resolution.
- Network connectivity to the API endpoint.
- Kubelet logs for registration or connection errors.

### Emergency or break-glass cluster access

In an emergency, normal user authentication may fail because of an identity-provider or authentication service problem.

A break-glass procedure is an alternate, tightly controlled way to obtain administrative access for recovery.

General principles:
- The procedure must be documented and approved by the organization.
- Access to the control-plane nodes must be restricted.
- The administrator kubeconfig must be protected as a highly privileged credential.
- Use the emergency kubeconfig only for recovery and investigation.
- Restore normal authentication and revoke or secure temporary access after the incident.

A privileged kubeconfig may be maintained on a control-plane node as part of the cluster installation and administration setup. The exact location and recovery process depend on how the cluster was installed and must be confirmed from the organization's runbook.

Do not copy kubeconfig files or credentials into a public repository or personal notes.

---

## 2.7 Lab — Kubernetes and OpenShift Command-Line Interfaces and APIs

### OpenShift lab tasks

1. Install `oc` and `kubectl` on the Linux workstation.
2. Verify both client versions.
3. Log in to the OpenShift cluster.
4. Verify the current user and API server.
5. List projects and namespaces.
6. Switch to a project.
7. List Pods, deployments and services.
8. Use `oc describe` to inspect a resource.
9. Use `oc explain` to inspect a resource definition.
10. List API resources and API versions.
11. Generate a YAML template using a client-side dry run.
12. Check the cluster version and ClusterOperators.
13. Review node status and recent events.
14. Inspect a Pod with an image pull error.
15. Review the node-level troubleshooting commands in a non-production lab.

---

## 2.8 Quiz — Kubernetes and OpenShift Command-Line Interfaces and APIs

1. What is the purpose of the `oc` CLI?
2. What is the difference between `oc` and `kubectl`?
3. Which OpenShift commands are not available in standard `kubectl`?
4. Where can the OpenShift CLI be downloaded?
5. Why should the client version be compatible with the cluster version?
6. What is the purpose of the `PATH` environment variable?
7. Which port is commonly used by the OpenShift API server?
8. How can a user authenticate using a token?
9. What is the difference between a regular user, system user and service account?
10. What is the relationship between an OpenShift Project and a Kubernetes Namespace?
11. What does `oc get all` display, and what does it omit?
12. What is the purpose of `oc describe`?
13. What is the purpose of `oc explain`?
14. How can API resource types and API versions be listed?
15. What does `Available=True` indicate for a ClusterOperator?
16. What does `Degraded=True` indicate?
17. Which commands can be used to investigate a Pod that is not starting?
18. How can a YAML template be generated without creating the resource?
19. What is the purpose of `oc debug node`?
20. Which services should be checked when a worker node is NotReady?
21. What information can be obtained from `journalctl`?
22. What is break-glass access, and why must it be controlled?

---

## 2.9 Summary

- `oc` is the primary command-line interface for OpenShift.
- `kubectl` supports standard Kubernetes operations, while `oc` also provides OpenShift-specific commands.
- The CLI can be installed on a jump host or Linux workstation with network access to the cluster.
- The Red Hat console and OpenShift web console provide CLI download options.
- A compatible CLI version should be used for the target cluster.
- CLI authentication can use a username/password or a token, depending on the configured identity provider.
- OpenShift supports regular users, system identities and service accounts.
- Projects are based on Kubernetes namespaces.
- `oc get` lists resources, `oc describe` provides detailed resource information and `oc explain` documents resource fields.
- `oc get all` does not show every resource type.
- `oc create -f` creates resources from YAML files; `--dry-run=client -o yaml` can generate a manifest for review.
- ClusterOperators provide important information about OpenShift platform health.
- A healthy operator is generally Available, not Progressing and not Degraded.
- Pod troubleshooting should include resource descriptions, events, logs and image-reference validation.
- Node troubleshooting may require `oc debug` or authorized SSH access, depending on node and cluster health.
- Kubelet and CRI-O are important services to inspect when investigating a node that is NotReady.
- Break-glass access is an emergency recovery mechanism and must be protected by strict access controls.

---

## 2.5 Assess the Health of an OpenShift Cluster — Extra Points

Use these commands and checks to build a basic understanding of API resources, Operators, cluster health and diagnostic information.

### Inspect the cluster version and API surface

```bash
# OpenShift version
oc get clusterversion

# Client and server versions
oc version

# API groups and versions
oc api-versions

# Available resource types, including API groups and short names
oc api-resources

# Resources that are namespaced
oc api-resources --namespaced=true

# Resources that are cluster-scoped
oc api-resources --namespaced=false
```

To inspect a specific resource definition:

```bash
oc explain pod
oc explain pod.spec
oc explain pod.spec.securityContext
oc explain deployment.spec
```

Use `oc explain` against the API version supported by the cluster rather than assuming every field exists in every release.

### Query Cluster Operators

Cluster Operators manage important OpenShift platform functions. The Cluster Version Operator (CVO) manages the lifecycle of the cluster Operators.

```bash
# List cluster operators
oc get clusteroperators

# Short form
oc get co

# Inspect one Operator in detail
oc describe clusteroperator <operator-name>

# View Operator YAML and conditions
oc get clusteroperator <operator-name> -o yaml
```

Review the `AVAILABLE`, `PROGRESSING` and `DEGRADED` conditions. A healthy Operator is normally Available=True, Progressing=False and Degraded=False. Investigate the message and related events when these conditions differ.

### Inspect resources using YAML and JSON

```bash
# YAML output
oc get pod <pod-name> -o yaml

# JSON output
oc get pod <pod-name> -o json

# Show selected fields with JSONPath
oc get node <node-name> \
 -o jsonpath='{.status.capacity.cpu}'

oc get node <node-name> \
 -o jsonpath='{.status.allocatable.cpu}'

# Number of allocatable Pods
oc get node <node-name> \
 -o jsonpath='{.status.allocatable.pods}'
```

YAML and JSON output are useful for detailed inspection, automation and scripting. JSONPath can extract selected values without printing the entire resource.

### Inspect node capacity and live resource usage

```bash
# Node CPU and memory usage
oc adm top nodes

# Pod CPU and memory usage in the current project
oc adm top pods

# Pod usage across all projects
oc adm top pods -A

# Detailed node information, including Capacity and Allocatable
oc describe node <node-name>
```

Understand the difference:

- **Capacity:** total resources reported by the node.
- **Allocatable:** resources available for Pods after system reservations and node-level overhead.
- **Usage:** current observed resource consumption, such as the values reported by `oc adm top`.

Do not treat `oc adm top` as the same thing as capacity or allocatable resources.

### Inspect Pod conditions and container resource usage

```bash
# Pod details and events
oc describe pod <pod-name> -n <namespace>

# Pod conditions and container status
oc get pod <pod-name> -n <namespace> -o yaml

# Resource usage
oc adm top pod <pod-name> -n <namespace> --containers
```

When using JSON output with `jq`, first inspect the structure and then select the required fields. Example:

```bash
oc get pod <pod-name> -n <namespace> -o json |
 jq '.status.conditions'
```

### Inspect events

```bash
# Events in the current project
oc get events

# Events in all projects
oc get events -A

# Sort by event timestamp
oc get events --sort-by=.lastTimestamp

# Events in a particular namespace
oc get events -n <namespace>
```

Events give a high-level view of scheduling, image pulls, container starts, restarts and other significant changes. Use Pod logs for more detailed application output.

### Collect diagnostic information with must-gather

`oc adm must-gather` collects cluster resource definitions and diagnostic information to help investigate cluster issues.

```bash
# Collect diagnostic data using the default image
oc adm must-gather

# Specify an output directory
oc adm must-gather --dest-dir=./must-gather-output
```

The command may require elevated cluster permissions and can collect sensitive operational data. Store and share the output according to organizational security policy.

### Authentication notes

The workbook emphasizes token authentication as a reliable method across environments where enterprise SSO may replace the normal console login page.

```bash
# Log in using a token
oc login https://api.<cluster-domain>:6443 \
 --token=<token>

# Display current identity
oc whoami

# Display current API server
oc whoami --show-server
```

Avoid placing passwords or tokens directly in shell history, screenshots or GitHub notes. Do not use plain-text passwords in production scripts or shell history.

### Chapter 2 completion checklist

- [ ] Explain `oc` versus `kubectl`.
- [ ] Log in using an approved authentication method.
- [ ] Identify the current user, project and API server.
- [ ] Create and switch between projects.
- [ ] List namespaced and cluster-scoped API resources.
- [ ] Inspect API groups, versions and resource schemas.
- [ ] Read resource YAML and JSON.
- [ ] Use JSONPath and `jq` to extract fields.
- [ ] Inspect Cluster Operator conditions.
- [ ] Check node capacity, allocatable resources and live usage.
- [ ] Inspect Pod conditions, container usage, logs and events.
- [ ] Collect diagnostic information with `oc adm must-gather`.

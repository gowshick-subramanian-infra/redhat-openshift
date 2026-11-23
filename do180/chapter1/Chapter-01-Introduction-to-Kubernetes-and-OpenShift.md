# Chapter 1 — Introduction to Kubernetes and OpenShift

> DO180 study notes
---

## 1.1 Containers and Kubernetes

### Why containers are used
- An application needs more than its source code to run. It also depends on libraries, runtime components, configuration and system-level dependencies.
- A container image packages an application and its required dependencies into a consistent unit.
- Containers help keep application environments consistent across development, testing and production.
- Containers are not the same as virtual machines. A VM includes a guest operating system; containers share the host operating-system kernel while isolating application processes.

### Kubernetes
- Kubernetes is a container-orchestration platform.
- It manages the application lifecycle after containers are packaged.
- It provides mechanisms to deploy and manage workloads across a cluster.
- A cluster consists of control-plane components and worker nodes.
- The control plane manages the cluster; worker nodes run application workloads.

### Pods
- A Pod is the basic deployable workload unit in Kubernetes.
- A Pod can contain one or more closely related containers.
- Application workloads are scheduled onto worker nodes.
- A Pod is not a permanent server. It can be recreated or scheduled on another node.

### Kubernetes and OpenShift
- OpenShift is Red Hat's enterprise Kubernetes platform.
- Kubernetes provides the orchestration foundation.
- OpenShift adds an integrated platform experience, supported operating-system integration, a web console, monitoring and additional platform capabilities.
- Kubernetes concepts and APIs remain important when working with OpenShift.

---

## 1.2 Matching Containers and Kubernetes

### Container runtime concepts
Several container technologies are used in container-based environments:

| Technology | Role |
|---|---|
| Podman | Tool for building, running, inspecting and troubleshooting containers |
| Docker | Container platform/tooling commonly used to build and run containers |
| CRI-O | Container runtime used by OpenShift to run Kubernetes workloads |
| containerd | Another container runtime used in Kubernetes environments |

### CRI-O and Podman
- OpenShift uses CRI-O as the runtime responsible for running containers for Kubernetes workloads.
- Podman may also be available on an OpenShift node, but it is not the runtime managing the cluster's Pods.
- Podman is useful for container inspection and troubleshooting tasks.
- `crictl`/CRI-O-level tooling and Podman serve different purposes; they should not be treated as interchangeable cluster-management interfaces.

### Important distinction
- **Kubernetes manages Pods and workload lifecycle.**
- **CRI-O runs the containers requested by Kubernetes.**
- **Podman is an independent container tool** that can be used for local container operations and investigation.

### Container and VM workloads
Application workloads can be grouped into:
- Traditional/standalone applications.
- Virtualized applications running in virtual machines.
- Containerized applications running as containers and Pods.

OpenShift can also support virtual-machine workloads when the OpenShift Virtualization capability is installed and configured.

---

## 1.3 Red Hat OpenShift Components and Editions

### OpenShift platform
OpenShift is more than an upstream Kubernetes installation. It combines Kubernetes with platform components and integrated operational capabilities.

Key OpenShift platform capabilities:

1. Integrated web console.
2. Administrator and Developer perspectives.
3. OpenShift Virtualization interface for VM-related operations.
4. Integrated cluster monitoring and observability.
5. Platform-managed operating-system lifecycle using Red Hat Enterprise Linux CoreOS (RHCOS).
6. OpenShift APIs and platform-specific resources.

### Administrator and Developer perspectives
- The **Administrator** perspective exposes cluster and platform administration functions.
- The **Developer** perspective focuses on application and project workflows.
- Access to functions depends on the user's role and permissions.
- Developers should not automatically receive cluster-administrator privileges.

### OpenShift Virtualization
- OpenShift can host virtual machines through OpenShift Virtualization.
- VM lifecycle operations include creating and managing virtual machines and related templates.
- This is an additional OpenShift capability; it is not a basic Kubernetes Pod feature.

### OpenShift deployment locations
OpenShift clusters can be deployed on-premises or on supported cloud infrastructure.
- In cloud deployments, the underlying infrastructure and region are selected as part of the infrastructure design.
- In on-premises deployments, cluster placement depends on the organization's data-centre design.
- High availability requires planning across failure domains and retaining enough spare capacity to tolerate failures.

### Editions and product variants

---

## 1.4 Matching Red Hat OpenShift Components and Editions

Use these relationships when revising:

| Component / capability | Main purpose |
|---|---|
| Kubernetes | Orchestrates containerized workloads |
| OpenShift | Red Hat's enterprise Kubernetes platform |
| OpenShift web console | Browser-based cluster and application interface |
| Administrator perspective | Cluster and platform administration |
| Developer perspective | Application and project workflows |
| OpenShift Virtualization | Runs and manages virtual machines on OpenShift |
| RHCOS | The operating system used for standard OpenShift cluster nodes |
| CRI-O | Runs containers for Kubernetes workloads |
| Prometheus-based monitoring | Collects and exposes cluster/workload metrics |

**Revision point:** Kubernetes is the orchestration foundation; OpenShift adds an integrated enterprise platform around it.

---

## 1.5 Navigate the OpenShift Web Console

### Accessing the console
- Open the OpenShift console URL in a browser.
- Authenticate using an account that has access to the cluster.
- The console provides an overview of cluster state and navigation to administration and application functions.

### Main perspectives
- **Administrator:** used for cluster-level operations and infrastructure visibility.
- **Developer:** used for project and application workflows.

### Useful areas to become familiar with
- Overview
- Projects / Namespaces
- Workloads
- Compute / Nodes
- Storage
- Networking
- Operators
- Observe / Monitoring
- Virtualization, when installed

### API URL and console URL
The distinguish the browser console URL from the Kubernetes API endpoint:
- The API endpoint commonly uses an `api.<cluster-domain>` hostname.
- The Kubernetes API server normally listens on TCP port `6443`.
- The console has a separate browser-facing URL.
- The exact hostnames are cluster-specific and should be obtained from the console or cluster administrator.

---

## 1.6 Guided Exercise — Navigate the OpenShift Web Console

The following workflow can be used to become familiar with the console and cluster overview.

### Practice checklist
- [ ] Open the OpenShift console.
- [ ] Sign in with an authorized account.
- [ ] Identify the Administrator perspective.
- [ ] Identify the Developer perspective.
- [ ] Open the cluster overview.
- [ ] Locate the API endpoint information.
- [ ] Find the Projects / Namespaces area.
- [ ] Locate Workloads, Compute, Storage and Networking.
- [ ] Check whether the Virtualization section is available.

---

## 1.7 Monitor an OpenShift Cluster

### Observability
Observability helps determine the current and historical condition of the cluster and its workloads.

The OpenShift monitoring interface provides visibility into:
- Cluster and workload status.
- Pod readiness and status.
- Node CPU and memory utilization.
- Alerts and their severity.
- Metrics over a selected time range.
- Graphs that can help identify periods when a workload stopped or restarted.

### What to check during a health review
1. **Cluster overview:** check the general cluster condition.
2. **Nodes:** review node availability and resource usage.
3. **Pods / workloads:** identify Pods that are not ready or are in an unexpected state.
4. **Storage:** check for storage-related warnings or errors.
5. **Alerts:** review warning, critical and informational alerts.
6. **Metrics:** inspect CPU and memory usage over time.
7. **Time range:** select a suitable period when investigating a past incident.

### Why historical metrics matter
A Pod may be healthy now but may have restarted earlier. Current status alone may not explain the incident. Historical metrics and events can help correlate a restart with resource pressure or another cluster event.

### Prometheus and dashboards
- OpenShift monitoring uses Prometheus and Prometheus Query Language (PromQL) for metrics collection and querying.
- Metrics can be filtered and viewed through dashboards.
- Dashboard time ranges can be adjusted, for example to inspect the previous 30 minutes, one hour or several hours.
- Detailed PromQL usage and dashboard customization are deferred in the.

---

## 1.8 Guided Exercise — Monitor an OpenShift Cluster

### Practice checklist
- [ ] Open the cluster Overview page.
- [ ] Review the number and condition of nodes.
- [ ] Review workload and Pod status.
- [ ] Identify Pods that are not ready or are failing.
- [ ] Open the Alerts area and review alert severity.
- [ ] Open metrics and inspect CPU and memory graphs.
- [ ] Change the time range and compare the graph.
- [ ] Investigate any gap or sudden change in a metric.

**Important:** A dashboard is a starting point for investigation. Use events, logs, resource descriptions and metrics together before deciding the root cause.

---

## 1.9 Lab — Introduction to Kubernetes and OpenShift

### Suggested OpenShift lab checklist
- [ ] Access the OpenShift console.
- [ ] Identify the API endpoint.
- [ ] Review the cluster overview.
- [ ] Identify control-plane and worker nodes.
- [ ] Review workload and Pod status.
- [ ] Review alerts and resource metrics.

Use a non-production OpenShift lab and only the credentials assigned to your own account.

---

## 1.10 Quiz — Introduction to Kubernetes and OpenShift

### Self-check questions
1. What problem does Kubernetes solve?
2. What is the difference between a container and a virtual machine?
3. What is a Pod?
4. Which part of the cluster manages workloads, and which part runs them?
5. What does OpenShift add around Kubernetes?
6. What is the purpose of the Administrator and Developer perspectives?
7. What is the role of CRI-O in OpenShift?
8. Is Podman the runtime responsible for managing OpenShift Pods?
9. What is RHCOS?
10. Why are historical metrics useful during incident investigation?
11. Which port is normally used by the Kubernetes API server?
12. Why should monitoring include nodes, Pods, storage and alerts?

---

## 1.11 Summary

- Kubernetes orchestrates containerized workloads.
- Pods are the basic deployable workload units.
- OpenShift is Red Hat's enterprise Kubernetes platform.
- OpenShift provides an integrated web console, role-oriented perspectives, monitoring and additional platform capabilities.
- CRI-O is the container runtime used by OpenShift for Kubernetes workloads.
- Podman is a separate container tool and is useful for inspection and troubleshooting.
- RHCOS is the standard operating system for OpenShift cluster nodes.
- Monitoring should include workload health, node resources, storage, alerts and historical metrics.
- Cluster availability and capacity must be designed around failure scenarios, not just normal operating conditions.

---

---

## Additional Notes — OpenShift Cluster Architecture

### Control-plane nodes

The control plane manages the OpenShift cluster and hosts the services that maintain the cluster's desired state.

Key responsibilities include:
- Exposing the Kubernetes API.
- Scheduling workloads.
- Maintaining cluster state.
- Running controllers and platform operators.
- Coordinating cluster configuration and health.

Control-plane nodes are infrastructure nodes. Application workloads are normally placed on worker nodes unless the cluster is intentionally configured for a different topology.

### Worker nodes

Worker nodes provide compute capacity for application workloads. They run the components required to register the node, start Pods, provide the container runtime and support Pod networking and storage access.

Pods are scheduled onto eligible nodes according to resource requests, scheduling constraints, node conditions and available capacity.

### High availability and etcd quorum

OpenShift uses etcd to store cluster state. Etcd requires a majority of its voting members (a quorum) to accept changes.

For `N` voting members, the majority is:

\[
\left\lfloor \frac{N}{2} \right\rfloor + 1
\]

| Etcd members | Majority required | Members that can fail while retaining quorum |
|---:|---:|---:|
| 1 | 1 | 0 |
| 3 | 2 | 1 |
| 5 | 3 | 2 |

An odd number of etcd voting members is generally preferred because adding an even-numbered member does not necessarily increase failure tolerance.

A common production control-plane topology uses three control-plane nodes. A single-node topology is useful for constrained, edge, development or testing scenarios, but it does not provide the same control-plane fault tolerance as a multi-node cluster.

### API endpoint and load balancing

Clients communicate with the cluster through a stable API endpoint. A highly available cluster commonly uses a load balancer and a virtual IP (VIP), so clients do not need to select an individual control-plane node.

The load balancer distributes eligible connections across healthy API server instances. HAProxy is one implementation used in OpenShift installation architectures.

Important terms:

- **API endpoint:** Stable address used by clients to reach the cluster API.
- **Virtual IP (VIP):** Address presented by the load-balancing layer.
- **Load balancer:** Distributes connections across healthy backend endpoints.
- **Health check:** Determines whether a backend should receive traffic.

The exact load-balancer implementation depends on the installation platform and cluster design.

### Single Node OpenShift (SNO)

Single Node OpenShift is a deployment topology in which one node hosts the control plane and runs workloads.

Typical use cases include:
- Edge locations.
- Remote sites with limited hardware.
- Development and testing.
- Environments that require a compact cluster footprint.

SNO has a single-node failure domain and should not be considered equivalent to a multi-node highly available control plane.

### CodeReady Containers (CRC)

CodeReady Containers (CRC) provides a local OpenShift environment intended primarily for development and testing on a workstation.

CRC and SNO are both single-node-oriented experiences, but they are not interchangeable terms. CRC provides a local developer environment, while SNO is an OpenShift cluster topology used in supported deployment scenarios.

### Cluster design considerations

When planning an OpenShift cluster, consider:
- Availability requirements and acceptable downtime.
- Control-plane quorum and failure tolerance.
- Worker-node count and sizing.
- Application CPU and memory requirements.
- Network and load-balancer design.
- Storage availability and persistence.
- Failure domains such as racks, availability zones or data centers.
- Backup, recovery and day-two operations.

Cluster installation is only the starting point. Production readiness also requires identity integration, monitoring, logging, backup and restore, security controls, storage configuration and operational procedures.

---

## 1.10 A Few More Points to Know

The following points give a little more background on OpenShift offerings, the web console and cluster monitoring.

### OpenShift offerings and responsibilities

| Offering / concept | What to understand |
|---|---|
| Kubernetes | Container orchestration foundation; does not by itself provide the complete OpenShift application platform |
| Red Hat OpenShift Container Platform | Enterprise application platform built on Kubernetes |
| Red Hat OpenShift Platform Plus | Extended OpenShift offering with additional platform capabilities |
| Red Hat OpenShift Local | Local OpenShift environment for learning, testing and exploration |
| Managed OpenShift | Red Hat or a cloud provider takes responsibility for more of the platform operation |
| Self-managed OpenShift | The organization has greater control and also takes on more installation and operational responsibility |
| Red Hat Enterprise Linux CoreOS (RHCOS) | Immutable, OpenShift-optimized operating system for cluster nodes |

Product packaging and names can change between releases. Confirm the exact entitlements and components for the cluster version in use.

### More web console areas to explore

The console is a separate OpenShift management interface that uses Kubernetes and OpenShift APIs. It is not the same as the optional Kubernetes Dashboard.

Practise locating the following areas:

- **Home → Overview:** cluster status and high-level information.
- **Home → Search:** find resources by kind, name or namespace.
- **Home → API Explorer:** inspect resource kinds, schemas and API metadata.
- **Workloads → Pods:** inspect Pod status, metrics, events, logs, YAML and terminal.
- **Compute → Nodes:** inspect node details, conditions, capacity, metrics, events and YAML.
- **Observe → Metrics:** query and graph metrics.
- **Observe → Alerting:** inspect firing alerts and their severity.
- **Networking / Storage:** inspect the cluster resources used by applications.
- **Operators → Installed Operators:** view installed operators and their provided capabilities.

Useful commands:

```bash
# Log in, then retrieve the web console URL
oc login https://api.<cluster-domain>:6443
oc whoami --show-console

# Show the API server endpoint
oc whoami --show-server
```

### Inspecting nodes and Pods from the console

For a node:
1. Open **Compute → Nodes**.
2. Select a node.
3. Review its status and conditions.
4. Inspect capacity and allocatable resources.
5. Review metrics, events and YAML.

For a Pod:
1. Open **Workloads → Pods**.
2. Select the project.
3. Open a Pod.
4. Review details, environment, events, logs, metrics, YAML and terminal.
5. Use the terminal for investigation, not as the normal way to make persistent application changes. Correct the workload configuration and redeploy instead.

### Basic monitoring concepts

OpenShift monitoring is based on Prometheus. Metrics can be exposed by applications through a `/metrics` endpoint when the application is instrumented.

- **Prometheus:** collects and stores time-series metrics.
- **PromQL:** query language used to select and calculate metrics.
- **ServiceMonitor:** defines how Prometheus discovers and scrapes metrics from a Service.
- **PodMonitor:** defines how Prometheus discovers and scrapes metrics directly from Pods.
- **Alerting:** evaluates configured conditions and raises alerts when thresholds or rules are met.
- **Events:** provide a concise record of significant changes or conditions.
- **Logs:** provide detailed application or component output for troubleshooting.

Example navigation:
- **Observe → Metrics** to run queries and view graphs.
- **Observe → Alerting** to review firing alerts.
- **Home → Events** to review cluster or project events.

A ServiceMonitor or PodMonitor is not automatically created for every application. The application must expose metrics and the monitoring configuration must select it.

### Cluster architecture — quorum clarification

The discuss three control-plane nodes and etcd quorum. Keep this distinction clear:

- etcd requires a majority of members to remain available to maintain quorum.
- A three-member etcd cluster needs two members and can tolerate one member failure.
- A five-member etcd cluster needs three members and can tolerate two member failures.
- Adding more members increases failure tolerance only when the members are distributed across suitable failure domains and network connectivity remains healthy.
- A load balancer or virtual IP provides a stable API endpoint; it does not replace etcd quorum.

The also discusses HAProxy, VIPs, Single Node OpenShift (SNO) and CodeReady Containers (CRC). Treat CRC as a historical product name; current local OpenShift learning environments should be checked against the current Red Hat product documentation.

### Chapter 1 completion checklist

- [ ] Explain why containers and Kubernetes are used.
- [ ] Explain the purpose of Pods and the control plane.
- [ ] Distinguish Kubernetes, OpenShift, RHCOS, CRI-O and Podman.
- [ ] Match the OpenShift offerings and managed/self-managed responsibility models.
- [ ] Navigate both console perspectives.
- [ ] Find nodes, Pods, workloads, operators and API resources in the console.
- [ ] View Pod and node logs, events, YAML and metrics.
- [ ] Explain Prometheus, PromQL, ServiceMonitor and PodMonitor.
- [ ] Locate metrics and alerts in the console.
- [ ] Explain etcd quorum and the purpose of an API load balancer.

# Chapter 3 — Run Applications as Containers and Pods

> DO180 study notes

## 3.1 Create Linux Containers and Kubernetes Pods

### Containers and images

A container is a running instance of a container image. An image packages an application, its runtime dependencies and metadata.

A container runtime creates and manages containers from images. On OpenShift, CRI-O is the Kubernetes container runtime. Podman is commonly used on a Linux workstation to build, inspect and run containers independently of the cluster.

### Container versus Pod

| Container | Pod |
|---|---|
| Runs an application process from an image | Smallest deployable unit in Kubernetes |
| Managed by a container runtime | Managed by Kubernetes |
| Has its own container configuration | Can contain one or more containers |
| Can run outside Kubernetes | Scheduled onto a cluster node |

Containers in the same Pod share the Pod network namespace and can communicate through `localhost`. A Pod can also define shared volumes for its containers.

### Run a container with Podman

Pull an image:

```bash
podman pull docker.io/library/httpd:latest
```

Run a container:

```bash
podman run -d --name web-test -p 8080:80 docker.io/library/httpd:latest
```

List running containers:

```bash
podman ps
```

List all containers, including stopped containers:

```bash
podman ps -a
```

View logs:

```bash
podman logs web-test
```

Stop and remove the container:

```bash
podman stop web-test
podman rm web-test
```

### Create a Pod in OpenShift

Create or select a project:

```bash
oc new-project pod-lab
oc project pod-lab
```

Create a Pod directly:

```bash
oc run web-pod --image=docker.io/library/httpd:latest
```

Check the Pod:

```bash
oc get pods
oc get pods -o wide
```

Inspect it:

```bash
oc describe pod web-pod
```

Delete the Pod:

```bash
oc delete pod web-pod
```

A Pod created directly with `oc run` is not automatically managed by a Deployment. For an application that should be recreated and maintained, use a workload controller such as a Deployment.

### Run a one-time command

A Pod can be created to execute a command and then exit:

```bash
oc run task-pod --image=registry.access.redhat.com/ubi9/ubi \
 --restart=Never -- ls -l /
```

Check its status and output:

```bash
oc get pods
oc logs task-pod
```

A one-time task may reach the `Completed` state after its command finishes. This is different from a long-running application Pod.

### Create a Deployment

A Deployment manages application Pods and maintains the desired replica count.

```bash
oc create deployment web-app \
 --image=docker.io/library/httpd:latest
```

Check the Deployment and Pods:

```bash
oc get deployments
oc get pods
```

Scale the Deployment:

```bash
oc scale deployment/web-app --replicas=3
```

Check the replica status:

```bash
oc get deployment web-app
oc get pods -o wide
```

Delete the Deployment:

```bash
oc delete deployment web-app
```

### Access a running container

View container logs:

```bash
oc logs <pod-name>
```

For a Pod with multiple containers:

```bash
oc logs <pod-name> -c <container-name>
```

Open a shell inside a running container, if the image includes one:

```bash
oc exec -it <pod-name> -- /bin/sh
```

Run a single command without opening a shell:

```bash
oc exec <pod-name> -- <command>
```

Not all minimal images contain `/bin/bash`; use a shell that exists in the image.

---

## 3.2 Guided Exercise — Create Linux Containers and Kubernetes Pods

### Container exercise

```bash
podman pull docker.io/library/httpd:latest
podman run -d --name httpd-test -p 8080:80 docker.io/library/httpd:latest
podman ps
podman logs httpd-test
```

Test the application from the Linux host:

```bash
curl http://localhost:8080
```

Clean up:

```bash
podman stop httpd-test
podman rm httpd-test
```

### Pod exercise

```bash
oc new-project pod-practice
oc run httpd-pod --image=docker.io/library/httpd:latest
oc get pods -o wide
oc describe pod httpd-pod
oc logs httpd-pod
oc delete pod httpd-pod
```

### Deployment exercise

```bash
oc create deployment httpd-app \
 --image=docker.io/library/httpd:latest

oc get deployment
oc get pods
oc scale deployment/httpd-app --replicas=2
oc get pods -o wide
```

---

## 3.3 Find and Inspect Container Images

### Image reference format

A container image reference commonly follows this pattern:

```text
[registry/]repository[:tag]
```

Examples:

```text
docker.io/library/httpd:latest
registry.access.redhat.com/ubi9/ubi:latest
quay.io/<organization>/<image>:<tag>
```

For predictable deployments, specify the registry, repository and version tag explicitly. If a registry or tag is omitted, the client uses default image-resolution rules.

### Search for images

```bash
podman search httpd
podman search nginx
```

Review the image source, publisher, description and available tags before selecting an image.

### Pull and list images

```bash
podman pull docker.io/library/httpd:latest
podman images
```

Inspect image metadata:

```bash
podman inspect docker.io/library/httpd:latest
```

Image inspection can show:
- Image ID and tags.
- Creation time.
- Entrypoint and command.
- Environment variables.
- Exposed ports.
- Image configuration.

### Image registries

A registry stores and distributes container images.

Common registry types:

- **Public registry:** Images can be pulled by users according to the registry's access rules.
- **Private registry:** Authentication and authorization restrict image access.
- **Enterprise registry:** May provide access controls, scanning, replication and retention.
- **OpenShift integrated registry:** Stores images for use by cluster workloads when configured and available.

A private image requires suitable credentials. Do not store registry passwords or access tokens in source code or Git repositories.

### Authenticate to a registry

```bash
podman login <registry-host>
```

Example:

```bash
podman login docker.io
```

Log out when required:

```bash
podman logout <registry-host>
```

Protect the container authentication file because it contains registry credentials.

### Image tags

A tag is a human-readable reference to an image version.

Examples:

```text
my-web-app:1.0
my-web-app:1.1
my-web-app:latest
```

Tag a local image for a registry:

```bash
podman tag localhost/my-web-app:latest \
 docker.io/<account-name>/my-web-app:1.0
```

Push it:

```bash
podman push docker.io/<account-name>/my-web-app:1.0
```

Pull it:

```bash
podman pull docker.io/<account-name>/my-web-app:1.0
```

Use version-specific tags for controlled deployments. `latest` is mutable and does not guarantee that the same image will be used every time.

### Red Hat container image sources

Red Hat publishes supported container images through its registry services. Select a base image according to:
- Application runtime requirements.
- Supported operating-system version.
- Security and vulnerability status.
- Image lifecycle and update policy.
- Required subscription or entitlement.

---

## 3.4 Guided Exercise — Find and Inspect Container Images

```bash
podman search httpd
podman pull docker.io/library/httpd:latest
podman images
podman inspect docker.io/library/httpd:latest
```

Before using an image, verify:
- The registry is trusted.
- The repository and tag are correct.
- The image is suitable for the application.
- The architecture is compatible with the cluster nodes.
- Required registry authentication is available.
- The image is maintained and updated.

---

## 3.5 Troubleshoot Containers and Pods

### Check Pod status

```bash
oc get pods
oc get pods -o wide
```

Common Pod states:

| Status | Meaning |
|---|---|
| `Pending` | Not yet scheduled or waiting for required resources |
| `ContainerCreating` | Container is being prepared |
| `ErrImagePull` | Image pull failed |
| `ImagePullBackOff` | Image pull failed and Kubernetes is retrying with backoff |
| `CrashLoopBackOff` | Container repeatedly starts and exits |
| `Running` | Pod is running; check readiness separately |
| `Completed` | Workload finished successfully |

### Describe a Pod

```bash
oc describe pod <pod-name>
```

Review:
- Pod conditions.
- Container state and restart count.
- Image reference.
- Node assignment.
- Volumes and mounts.
- Events at the bottom of the output.

Events often provide the first useful clue when a Pod cannot start.

### View container logs

```bash
oc logs <pod-name>
```

For a multi-container Pod:

```bash
oc logs <pod-name> -c <container-name>
```

View logs from the previous container instance:

```bash
oc logs <pod-name> --previous
```

Follow logs continuously:

```bash
oc logs -f <pod-name>
```

### Execute a command inside a container

```bash
oc exec <pod-name> -- <command>
```

Interactive shell:

```bash
oc exec -it <pod-name> -- /bin/sh
```

### Troubleshoot image pull failures

If a Pod shows `ErrImagePull` or `ImagePullBackOff`, inspect it:

```bash
oc describe pod <pod-name>
```

Check:
1. Registry hostname.
2. Repository and image name.
3. Image tag or digest.
4. Whether the image exists.
5. Registry reachability from the cluster.
6. Whether authentication is required.
7. Whether the Pod or its service account can access the required image pull secret.

For private registries, configure an appropriate pull secret and associate it with the workload or service account according to the cluster's access policy.

### Troubleshoot CrashLoopBackOff

A container can enter `CrashLoopBackOff` when its process repeatedly exits.

```bash
oc describe pod <pod-name>
oc logs <pod-name>
oc logs <pod-name> --previous
```

Check for:
- Application startup errors.
- Incorrect command or arguments.
- Missing environment variables.
- Missing configuration files.
- Permission problems.
- Incorrect ports or dependencies.
- Health probe failures.
- Resource constraints.

A Pod in `Running` state is not necessarily ready to serve traffic. Check readiness conditions as well.

### Inspect Pod placement and events

```bash
oc get pods -o wide
oc get events --sort-by=.lastTimestamp
```

Pod IP addresses are internal and can change when Pods are recreated. Use Services and Routes for stable application access rather than relying on a Pod IP.

### Container runtime tools

OpenShift uses CRI-O as its Kubernetes container runtime. Podman is a separate tool commonly used to build and run containers outside the cluster.

`crictl` can inspect containers and Pods through the configured CRI runtime endpoint when it is available and supported on the node.

```bash
sudo crictl pods
sudo crictl ps
sudo crictl images
sudo crictl inspect <container-id>
sudo crictl logs <container-id>
```

Use node-level runtime commands only for authorized troubleshooting. Prefer `oc` commands for normal cluster administration.

---

## 3.6 Guided Exercise — Troubleshoot Containers and Pods

Use the following sequence when a Pod does not start as expected:

```bash
oc get pods -o wide
oc describe pod <pod-name>
oc logs <pod-name>
oc logs <pod-name> --previous
oc get events --sort-by=.lastTimestamp
```

For a multi-container Pod:

```bash
oc logs <pod-name> -c <container-name>
```

For a running container:

```bash
oc exec -it <pod-name> -- /bin/sh
```

Identify whether the issue is related to image retrieval, scheduling, configuration, application startup, permissions, storage or resource availability.

---

## Additional Practical Notes — Build and Publish a Custom Image

### Containerfile / Dockerfile

A `Containerfile` or `Dockerfile` contains instructions used to build a container image.

Example:

```dockerfile
FROM registry.access.redhat.com/ubi9/ubi
RUN dnf -y install httpd && dnf clean all
RUN echo "Hello from my OpenShift application" > /var/www/html/index.html
EXPOSE 8080
CMD ["httpd", "-D", "FOREGROUND"]
```

This is a learning example. For production, use a supported base image and follow its documented runtime and non-root requirements.

### Build an image with Podman

From the directory containing the `Containerfile` or `Dockerfile`:

```bash
podman build -t my-web-app:1.0.
```

List the image:

```bash
podman images
```

### Run and test locally

```bash
podman run -d --name my-web-app -p 8080:8080 my-web-app:1.0
podman ps
podman logs my-web-app
curl http://localhost:8080
```

Stop and remove the test container:

```bash
podman stop my-web-app
podman rm my-web-app
```

### Tag and push to a registry

```bash
podman login <registry-host>
podman tag my-web-app:1.0 <registry-host>/<account-or-project>/my-web-app:1.0
podman push <registry-host>/<account-or-project>/my-web-app:1.0
```

Confirm that the image exists in the registry and that the OpenShift cluster can access it.

### Deploy a custom image to OpenShift

```bash
oc new-project image-demo
oc create deployment my-web-app \
 --image=<registry-host>/<account-or-project>/my-web-app:1.0
```

Check the Deployment and Pod:

```bash
oc get deployments
oc get pods
oc describe pod <pod-name>
```

For private registries, configure the required image pull secret.

### OpenShift integrated image registry

OpenShift provides an integrated image registry that can store images for cluster workloads when it is configured and available.

Typical workflow:
1. Build the image.
2. Authenticate to the registry.
3. Tag the image with the registry address and target repository/project.
4. Push the image.
5. Reference the image from an OpenShift workload.

The registry endpoint, authentication method and repository naming depend on cluster configuration. Obtain the correct values from the cluster administrator or OpenShift console rather than assuming a fixed hostname.

### ImageStreams

An OpenShift ImageStream is an API resource that provides an abstraction for referring to and tracking container images.

ImageStreams can:
- Reference images in external registries.
- Track image tags.
- Provide a stable OpenShift resource name for image references.
- Participate in image change-trigger workflows.

List ImageStreams:

```bash
oc get imagestreams
```

Inspect an ImageStream:

```bash
oc describe imagestream <imagestream-name>
```

An ImageStream is not the image binary itself. It maintains image references and related metadata in OpenShift.

---

## 3.7 Lab — Run Applications as Containers and Pods

1. Search for a suitable image using Podman.
2. Pull the image from a registry.
3. List and inspect local images.
4. Run a container locally and validate the application.
5. Create a Pod in OpenShift using an existing image.
6. Inspect the Pod status, IP address and node assignment.
7. Review Pod events and logs.
8. Execute a command inside a running container.
9. Create a simple `Containerfile` or `Dockerfile`.
10. Build a custom image with Podman.
11. Test the image locally.
12. Tag and push the image to a registry.
13. Deploy the custom image to OpenShift.
14. Troubleshoot image-pull and application-startup failures.
15. Review the integrated registry and ImageStream resources.

## 3.8 Quiz — Run Applications as Containers and Pods

1. What is a container image?
2. What information is represented in an image reference?
3. What is the purpose of a container registry?
4. What is the difference between a public and private registry?
5. Which Podman command searches for images?
6. Which command downloads an image locally?
7. How can image metadata be inspected?
8. What is the purpose of an image tag?
9. What is the difference between building an image and running a container?
10. Why should a custom image be tested locally before publishing it?
11. Why must an image be pushed to a registry before a remote cluster can pull it?
12. What does `ImagePullBackOff` indicate?
13. Which command provides detailed Pod events and status?
14. How can logs from the previous container instance be viewed?
15. How can a command be executed inside a running container?
16. What is the role of CRI-O in OpenShift?
17. How is Podman different from CRI-O?
18. What is the purpose of `crictl`?
19. What is an OpenShift ImageStream?
20. Why must registry credentials and pull secrets be protected?

## 3.9 Summary

- Containers are created from images, which are normally retrieved from registries.
- Podman can search, pull, inspect, build, run, tag and push images.
- A private registry requires suitable authentication and authorization.
- Version-specific image tags support more controlled deployments than mutable tags such as `latest`.
- A custom image can be built from a `Containerfile` or `Dockerfile`.
- Test a custom image locally before publishing it.
- OpenShift must be able to reach and authenticate to a registry to pull an image.
- The integrated registry can store images for cluster workloads when configured.
- ImageStreams provide an OpenShift resource for tracking image references and tags.
- `oc get`, `oc describe`, `oc logs` and `oc get events` are key Pod troubleshooting commands.
- `ImagePullBackOff` commonly points to an image reference, registry connectivity or authentication problem.
- `CrashLoopBackOff` means the container repeatedly exits; inspect current and previous logs and Pod events.
- OpenShift uses CRI-O as its container runtime. Podman is commonly used for local image building and testing.
- `crictl` is a node-level tool for inspecting the CRI runtime and should be used only for authorized troubleshooting.

---

## 3.10 Additional DO180 Reference — Container and Pod Fundamentals

### Container lifecycle versus Pod lifecycle

A container is a running process created from an image. Kubernetes does not normally manage an individual container as the top-level workload; it manages Pods.

A Pod:
- Is the smallest deployable unit in Kubernetes.
- Can contain one or more containers that share network and storage context.
- Receives a Pod IP when scheduled and started.
- Can be recreated when a controller or administrator replaces it.
- Should be treated as replaceable rather than as a permanent host.

The Chapter 3 goal is to run and troubleshoot unmanaged Pods. Managed application rollouts and workload controllers are developed further in Chapter 4.

### Create a simple Pod from an image

```bash
# Create a Pod directly from an image
oc run web-test --image=nginx:latest

# Check Pod status
oc get pods

# Get additional placement and IP details
oc get pods -o wide

# Inspect the Pod
oc describe pod web-test
```

A directly created Pod is not automatically recreated by a Deployment if it is deleted. Use a workload controller when the application needs managed lifecycle and replica handling.

### Podman image and container commands

These commands cover the basic workflow of pulling an image, running a container, checking it, stopping it and removing it.

```bash
# Search for an image
podman search httpd

# Pull an image
podman pull docker.io/library/httpd

# List local images
podman images

# Inspect image metadata
podman inspect docker.io/library/httpd

# Run a container in detached mode
podman run -d --name web-test docker.io/library/httpd

# List running containers
podman ps

# List all containers
podman ps -a

# View container logs
podman logs web-test

# Stop and remove the container
podman stop web-test
podman rm web-test

# Remove a local image
podman rmi docker.io/library/httpd
```

The exact image name and tag should be selected for the intended application. Avoid using a mutable `latest` tag for reproducible production deployments.

### Image registries and private registry authentication

A registry stores and distributes container images. Companies may use private registries so that internal application images are stored and accessed within controlled environments.

Examples mentioned in the include:
- Docker Hub (`docker.io`)
- Red Hat registries
- Quay (`quay.io`)
- JFrog Artifactory
- Sonatype Nexus

The exact products available in an organization depend on its platform and licensing choices.

A private registry usually requires authentication. OpenShift workloads need an image pull secret with credentials authorized to pull the requested repository.

Example workflow (the secret name, registry and credentials are environment-specific):

```bash
# Create a registry pull secret
oc create secret docker-registry registry-credentials \
 --docker-server=<registry-host> \
 --docker-username=<username> \
 --docker-password=<password> \
 --docker-email=<email>

# Link the pull secret to the default service account
oc secrets link default registry-credentials --for=pull
```

Do not put real credentials in source-controlled YAML or public repositories. Follow the organization's preferred secret-management process.

### Build, tag and push a custom image

```bash
# Build from a Containerfile or Dockerfile in the current directory
podman build -t my-web-app:1.0.

# Inspect locally available images
podman images

# Tag for a registry repository
podman tag my-web-app:1.0 \
 <registry-host>/<repository>/my-web-app:1.0

# Authenticate to the registry
podman login <registry-host>

# Push the image
podman push <registry-host>/<repository>/my-web-app:1.0
```

The 's practical task follows this general sequence: prepare a Linux VM, install Podman, create a Containerfile/Dockerfile, build and tag an image, push it to a registry, and create an OpenShift Pod using that image.

### Troubleshooting checklist

| Symptom | First checks 
|---|---|
| `ErrImagePull` | Image name, tag, registry hostname, network reachability and credentials |
| `ImagePullBackOff` | Pod events, pull secret, service-account linkage and registry permissions |
| `Pending` | Scheduling events, available node resources, storage claims and scheduling constraints |
| `CrashLoopBackOff` | Current and previous logs, startup command, configuration, permissions and probe failures |
| Pod is Running but not Ready | Readiness probe, application listener and container logs |

Useful commands:

```bash
oc get pods -o wide
oc describe pod <pod-name>
oc get events --sort-by=.lastTimestamp
oc logs <pod-name>
oc logs <pod-name> --previous
oc logs <pod-name> -c <container-name>
oc exec -it <pod-name> -- /bin/sh
```

For OpenShift, `oc rsh <pod-name>` is also available for opening a remote shell in a running Pod. `oc exec` is the Kubernetes-compatible command and is useful when working across Kubernetes and OpenShift.

### Chapter 3 completion checklist

- [ ] Search for and pull container images.
- [ ] List and inspect local images.
- [ ] Run, inspect, stop and remove a local container with Podman.
- [ ] Explain the difference between an image, a container and a Pod.
- [ ] Create and inspect an unmanaged Pod in OpenShift.
- [ ] Explain public and private registries.
- [ ] Build, tag and push a custom image.
- [ ] Configure image pull credentials for a private registry.
- [ ] Inspect Pod status, conditions and events.
- [ ] Retrieve current and previous container logs.
- [ ] Execute a command inside a running container.
- [ ] Diagnose common image-pull and startup failures.


# On-premise Installation Process

This document defines the recommended installation process for deploying Stream Orchestrator on multiple prepared Linux nodes. A node may be either a virtual machine or a physical server.

The intended reader is an engineer or AI assistant working with this repository. Use this document as the baseline installation strategy unless a later architecture decision explicitly replaces it.

## Assumptions

- The target environment consists of multiple Linux nodes, either virtual machines or physical servers.
- Linux is already installed on every node.
- The installation engineer can access the nodes remotely over SSH.
- The nodes have internet access during installation.
- Kubernetes will be installed as a self-managed cluster on the prepared Linux nodes.
- Stream Orchestrator will be deployed onto Kubernetes.
- Media server instances are deployed and managed by the orchestrator platform, not treated as externally managed infrastructure.

## Process Overview

The installation process is divided into one prerequisite phase and four execution phases.

| Order | Phase | Primary Tool | Purpose |
| --- | --- | --- | --- |
| 0 | Prerequisites | Customer or infrastructure team | Prepare Linux nodes for remote installation |
| 1 | Node Bootstrap | Ansible | Configure each Linux node as a Kubernetes-ready node |
| 2 | Kubernetes Cluster Provisioning | kubeadm + Ansible | Build the Kubernetes control plane and join worker nodes |
| 3 | Cluster Add-ons Installation | Helm | Install platform-level Kubernetes components |
| 4 | Orchestrator Deployment | Helm Chart | Install Stream Orchestrator |

## 0. Prerequisites

Node provisioning is treated as a prerequisite, not as part of the orchestrator installation process.

The reason is operational: the installation engineer works remotely. If Linux is not installed and reachable, the engineer cannot reliably execute the remaining installation steps.

The following conditions must be satisfied before the orchestrator installation starts:

- Linux OS is installed on every node.
- Each node is reachable through the network.
- SSH access is available.
- The installation account has passwordless or documented sudo access.
- Hostnames and IP addresses are assigned.
- The nodes can reach the internet for package and container image downloads.
- Time synchronization is available through NTP or chrony.

For physical servers, hardware provisioning and OS installation are intentionally outside the scope of this process. Technologies such as PXE, MAAS, Kickstart, or Ubuntu autoinstall may be used by the customer or infrastructure team, but they are not required implementation details for the orchestrator installer.

For virtual machines, VM creation and base image provisioning are also outside the scope of this process. Technologies such as Terraform, cloud-init, VMware templates, OpenStack images, or other virtualization tooling may be used before this process starts.

## 1. Node Bootstrap

Node Bootstrap prepares each Linux node so it can safely join a Kubernetes cluster.

Use **Ansible** for this phase.

Ansible is the preferred tool because it provides repeatable host configuration, inventory-based targeting, idempotent execution, and readable operational history.

### Responsibilities

This phase should configure:

- Linux users and SSH access required by the installer.
- Hostname and host-to-IP resolution.
- Package repositories.
- Kernel modules required by Kubernetes networking.
- `sysctl` settings required by Kubernetes.
- Swap disablement.
- Firewall rules.
- Time synchronization.
- Disk and volume mount points required by the platform.
- Container runtime installation and configuration.
- Kubernetes package installation, including `kubeadm`, `kubelet`, and `kubectl`.

### Recommended Technology Choices

| Concern | Recommended Choice |
| --- | --- |
| Automation | Ansible |
| Container runtime | containerd |
| Time synchronization | chrony |
| Network configuration | OS-native tooling such as Netplan or NetworkManager |
| Disk management | LVM when logical volume management is required |
| Firewall | firewalld, ufw, or nftables depending on the Linux distribution |

### Notes

Because internet access is available, the installer may use official OS package repositories, Kubernetes package repositories, Helm repositories, and public container registries.

For production installations, pin package versions where possible. The installer should avoid unbounded latest-version installs for Kubernetes, containerd, and critical add-ons.

## 2. Kubernetes Cluster Provisioning

Kubernetes Cluster Provisioning creates the Kubernetes cluster across the prepared Linux nodes.

Use **kubeadm + Ansible** for this phase.

`kubeadm` is the preferred Kubernetes bootstrap tool because it is standard, well documented, and keeps the cluster close to upstream Kubernetes behavior. Ansible should orchestrate the commands and configuration across nodes.

### Responsibilities

This phase should configure:

- Kubernetes control-plane initialization.
- Worker node join operations.
- Kubernetes version selection.
- Pod CIDR and Service CIDR.
- Container runtime integration.
- kubelet configuration.
- Control-plane endpoint configuration.
- Kubernetes admin kubeconfig collection.
- Basic cluster validation.

### High Availability

For production environments, the control plane should be highly available.

Recommended options:

- **kube-vip** for a Kubernetes-native virtual IP control-plane endpoint.
- **HAProxy + Keepalived** when the environment prefers traditional Linux load-balancing components.

The chosen control-plane endpoint must be decided before running `kubeadm init`.

### CNI Boundary

The Container Network Interface should be treated as a required cluster add-on.

The cluster will not be fully usable until a CNI is installed, but keeping CNI installation in the add-ons phase makes the boundary clearer:

- kubeadm creates the cluster.
- Helm installs the networking implementation.

Recommended CNI choices:

- **Cilium** for modern networking, observability, and network policy support.
- **Calico** for a widely adopted and straightforward Kubernetes networking setup.

## 3. Cluster Add-ons Installation

Cluster Add-ons Installation installs platform-level Kubernetes components required before the orchestrator application can run.

Use **Helm** for this phase.

Helm is the preferred tool because most Kubernetes platform components are distributed as Helm charts, and values files provide a clear way to manage environment-specific configuration.

### Responsibilities

This phase should install and configure:

- CNI.
- StorageClass provider.
- Ingress controller.
- LoadBalancer implementation for self-managed Kubernetes.
- Certificate management.
- Metrics server.
- Monitoring stack.
- Logging stack, if required.
- Image pull secrets, if private registries are used.

### Recommended Technology Choices

| Concern | Recommended Choice |
| --- | --- |
| Package manager | Helm |
| CNI | Cilium or Calico |
| Self-managed LoadBalancer | MetalLB for physical or simple VM environments; platform LoadBalancer integration when available |
| Ingress controller | ingress-nginx |
| Certificate management | cert-manager |
| StorageClass | Longhorn for simple operations, Rook Ceph for larger HA storage needs |
| Metrics | metrics-server |
| Monitoring | kube-prometheus-stack |
| Logging | Loki stack when centralized log collection is required |

### Installation Management

For a simple installer, Ansible may run Helm commands directly.

For a more mature operations model, prefer one of the following:

- **Helmfile** for ordered, repeatable Helm releases.
- **Argo CD** for GitOps-based lifecycle management after the initial cluster is ready.

The default recommendation is:

- Use Ansible to install Helm and bootstrap the first add-ons.
- Use Helm values files for all add-on configuration.
- Introduce Argo CD when continuous operations and declarative drift management are required.

## 4. Orchestrator Deployment

Orchestrator Deployment installs Stream Orchestrator onto the prepared Kubernetes platform.

Use a dedicated **Helm Chart** for this phase.

The Helm chart is the application packaging boundary. Environment-specific behavior should be expressed through values files, not by editing Kubernetes manifests directly.

The default installation model assumes that media server instances are under orchestrator control. The orchestrator must be able to provision, configure, scale, monitor, and replace media server instances at runtime. Externally managed media servers are out of scope for the default installation process because they limit runtime scale-out, scheduling, failure recovery, and state reconciliation.

### Responsibilities

This phase should install and configure:

- Orchestrator API service.
- Stream provisioning components.
- Stream gateway components.
- Database dependencies or external database connection settings.
- Media server deployment and runtime management settings.
- Media server scaling, scheduling, service exposure, and runtime configuration policies.
- Service accounts and RBAC.
- ConfigMaps and Secrets.
- Services.
- Ingress rules.
- Persistent volume claims.
- Horizontal or vertical scaling values.
- Health checks and readiness probes.

### Recommended Technology Choices

| Concern | Recommended Choice |
| --- | --- |
| Application packaging | Helm Chart |
| Environment configuration | values files |
| Secret integration | Kubernetes Secrets initially, External Secrets Operator for mature environments |
| Ingress exposure | ingress-nginx rules managed by chart values |
| TLS | cert-manager-managed certificates where available |
| Deployment operation | `helm upgrade --install` for initial installs, Argo CD for GitOps operations |

### Values Strategy

The chart should support separate values files per environment.

Recommended structure:

```text
deploy/
  helm/
    orchestrator/
      Chart.yaml
      values.yaml
      templates/
  environments/
    dev-values.yaml
    staging-values.yaml
    prod-values.yaml
```

The base `values.yaml` should contain safe defaults. Environment files should override only the settings that differ per installation.

## Recommended End-to-End Flow

The preferred installation flow is:

1. Confirm prerequisite Linux nodes are reachable over SSH.
2. Run Ansible Node Bootstrap playbooks.
3. Run Ansible-managed kubeadm cluster provisioning.
4. Install cluster add-ons with Helm.
5. Validate Kubernetes storage, ingress, DNS, and metrics behavior.
6. Deploy Stream Orchestrator with its Helm chart.
7. Validate orchestrator API, stream provisioning, stream gateway, and media path behavior.

## Best Practices

- Keep node provisioning and OS installation outside the orchestrator installer scope.
- Keep all host-level configuration in Ansible.
- Keep Kubernetes cluster bootstrap in kubeadm, orchestrated by Ansible.
- Keep platform components installable through Helm charts.
- Keep application deployment isolated in the orchestrator Helm chart.
- Pin Kubernetes and critical add-on versions for production.
- Store environment-specific configuration in values files.
- Avoid manually editing live Kubernetes resources after installation.
- Make every installation step repeatable and idempotent where possible.
- Capture validation commands for every phase.

## Phase Ownership Summary

| Phase | Owned By | Included in Orchestrator Installer |
| --- | --- | --- |
| Node provisioning and OS installation | Customer or infrastructure team | No |
| Node Bootstrap | Installation engineer | Yes |
| Kubernetes Cluster Provisioning | Installation engineer | Yes |
| Cluster Add-ons Installation | Installation engineer | Yes |
| Orchestrator Deployment | Installation engineer or platform operator | Yes |

## Future Documentation

This process document should eventually be complemented by:

- An Ansible inventory example.
- Node Bootstrap playbook documentation.
- kubeadm configuration examples.
- Helm add-on values examples.
- Orchestrator Helm chart values reference.
- End-to-end installation runbook.
- Post-installation validation checklist.

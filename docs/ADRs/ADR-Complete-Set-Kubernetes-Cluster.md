# Kubernetes Cluster Architecture Decision Records (ADRs)

This document contains the complete set of Architecture Decision Records
(ADR-001 through ADR-016) grounded in the Kubernetes cluster inventory.

## ADR-001 --- Kubernetes Cluster Topology

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The Kubernetes cluster is distributed across multiple infrastructure
locations and contains three currently operational Kubernetes nodes plus
one historical/unavailable node.

**Current topology:**

  -------------------------------------------------------------------------------------
  **Node**        **Location**   **Architecture**   **Role**             **State**
  --------------- -------------- ------------------ -------------------- --------------
  gcp-na-00       GCP / NA       amd64              control-plane + etcd Ready

  oci-sea-00      OCI / SEA      arm64              control-plane + etcd Ready

  onprem-sea-00   On-prem / SEA  arm64              worker               Ready

  euserv-eu-00    EU             ---                external-etcd-only   NotReady
  -------------------------------------------------------------------------------------

The cluster therefore combines cloud and on-prem infrastructure and
multiple CPU architectures.

### Decision

The Kubernetes cluster is operated as a distributed, multi-location
Kubernetes cluster with:

-   **two active control-plane/etcd nodes:** gcp-na-00, oci-sea-00

-   **one active worker node:** onprem-sea-00

-   **one historical/unavailable external-etcd node:** euserv-eu-00

The node inventory and role assignments above are considered the
authoritative description of the current cluster topology.

### Consequences

-   Cluster architecture spans infrastructure providers and locations.

-   Control-plane and workload placement cannot be assumed to be
    > homogeneous.

-   CPU architecture must be considered when scheduling workloads.

-   The unavailable euserv-eu-00 must be distinguished from active
    > cluster nodes in architectural discussions.

## ADR-002 --- Control-Plane Placement

**Status:** Accepted\
**Date:** 2026-08-26

### Context

gcp-na-00 and oci-sea-00 both operate Kubernetes control-plane
components and etcd.

Both nodes have:

-   node-role.kubernetes.io/control-plane=

-   node-role.kubernetes.io/etcd=

and both have:

-   node-role.kubernetes.io/control-plane:NoSchedule

The cluster therefore explicitly prevents ordinary workloads from being
scheduled onto control-plane nodes unless those workloads tolerate the
control-plane taint.

Several cluster-critical components currently tolerate this taint.

### Decision

Control-plane nodes are treated as dedicated control-plane
infrastructure by default.

The following nodes are control-plane nodes:

-   **gcp-na-00**

-   **oci-sea-00**

The standard control-plane taint remains the mechanism used to prevent
ordinary workloads from being scheduled there.

Cluster-system workloads may explicitly tolerate the control-plane taint
when their placement requires or permits execution on control-plane
nodes.

### Consequences

-   Ordinary application workloads do not automatically consume
    > control-plane capacity.

-   System workloads can intentionally run on control-plane nodes.

-   Scheduling behavior must be evaluated using both node taints and pod
    > tolerations.

-   A pod running on a control-plane node is not, by itself, evidence of
    > incorrect scheduling.

## ADR-003 --- Worker Placement and Workload Scheduling

**Status:** Accepted\
**Date:** 2026-08-26

### Context

onprem-sea-00 is the only active node designated exclusively as a
worker.

It has:

-   node-role.kubernetes.io/worker=

-   and no taints.

The active cluster currently has relatively few workload pods, while the
control-plane nodes host the Kubernetes control-plane and selected
system workloads.

### Decision

onprem-sea-00 is the cluster\'s dedicated general-purpose worker node.

Application workloads should be considered worker workloads by default.

Control-plane nodes are not considered general-purpose application
capacity unless a workload explicitly tolerates the relevant
control-plane taint.

### Consequences

-   Worker capacity is concentrated on onprem-sea-00.

-   Workload availability is inherently constrained by the number of
    > active worker nodes.

-   Pod placement must account for the fact that the worker and
    > control-plane pools are distinct.

## ADR-004 --- Node Taints, Labels and Scheduling Policy

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The cluster uses node labels and taints to express scheduling intent.

**Current relevant configuration:**

-   **oci-sea-00:** node-role.kubernetes.io/control-plane:NoSchedule

-   **gcp-na-00:** node-role.kubernetes.io/control-plane:NoSchedule,
    > low-resource=true:NoSchedule

-   **onprem-sea-00:** no taints, node-role.kubernetes.io/worker=

-   **euserv-eu-00:** node-role.kubernetes.io/etcd:NoSchedule,
    > node.kubernetes.io/unschedulable:NoSchedule,
    > node.kubernetes.io/unreachable:NoSchedule,
    > node.kubernetes.io/unreachable:NoExecute

### Decision

Node taints and labels are part of the cluster\'s scheduling contract.

Specifically:

-   node-role.kubernetes.io/control-plane:NoSchedule identifies
    > control-plane capacity as non-general-purpose capacity.

-   low-resource=true:NoSchedule identifies gcp-na-00 as unsuitable for
    > workloads that do not explicitly tolerate its resource constraint.

-   node-role.kubernetes.io/worker identifies onprem-sea-00 as worker
    > capacity.

-   Node readiness and lifecycle taints identify nodes that are
    > unavailable for normal scheduling.

Pod scheduling decisions must therefore be evaluated against the
complete combination of:

-   node labels,

-   node taints,

-   pod tolerations,

-   node selectors,

-   affinity/anti-affinity,

-   resource requests.

### Consequences

-   A pod\'s current node placement cannot be explained by node labels
    > alone. Its scheduling constraints must be inspected together with
    > node taints and available resources.

## ADR-005 --- etcd Topology and Failure Domains

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The cluster inventory identifies:

-   **gcp-na-00:** control-plane + etcd

-   **oci-sea-00:** control-plane + etcd

-   **euserv-eu-00:** external-etcd-only, NotReady

euserv-eu-00 has been unavailable since July 2026 and has:

-   Ready: Unknown

-   Kubelet stopped posting node status

-   Unschedulable: true

-   It has no running pods.

The inventory therefore distinguishes an active control-plane/etcd
population from an unavailable historical etcd node.

### Decision

The active Kubernetes control-plane/etcd topology is recorded as:

-   **gcp-na-00**

-   **oci-sea-00**

euserv-eu-00 is recorded as an unavailable external-etcd-only node, not
as an active Kubernetes compute node.

The existence of the Kubernetes Node object for euserv-eu-00 is not
treated as evidence that the node is operational.

### Consequences

-   The active control-plane/etcd topology is explicitly documented.

-   euserv-eu-00 remains relevant to historical cluster state and
    > lifecycle discussions.

-   Any future analysis of etcd quorum or failure tolerance must
    > distinguish active etcd membership from Kubernetes Node objects.

## ADR-006 --- Cluster Networking with Flannel

**Status:** Accepted\
**Date:** 2026-08-26

### Context

All active Kubernetes nodes run Flannel:

ghcr.io/flannel-io/flannel:v0.28.4

Each node has a dedicated PodCIDR:

-   **gcp-na-00:** 198.18.3.0/24

-   **oci-sea-00:** 198.18.4.0/24

-   **onprem-sea-00:** 198.18.1.0/24

-   **euserv-eu-00:** 198.18.2.0/24

The nodes report:

-   NetworkUnavailable: False

-   Reason: FlannelIsUp

### Decision

Flannel is the cluster\'s Kubernetes pod networking implementation.

The cluster uses per-node PodCIDR allocation with Flannel as the
networking layer.

### Consequences

-   Pod networking depends on Flannel being operational on each active
    > node.

-   Pod IPs are allocated from node-associated PodCIDRs.

-   Node networking and Pod networking are distinct from the nodes\'
    > 100.96.0.x internal addresses.

## ADR-007 --- Pod CIDR Allocation

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The cluster currently assigns individual /24 PodCIDRs to nodes.

**Observed allocation:**

  -----------------------------------------------------------------------
  **Node**                            **PodCIDR**
  ----------------------------------- -----------------------------------
  onprem-sea-00                       198.18.1.0/24

  euserv-eu-00                        198.18.2.0/24

  gcp-na-00                           198.18.3.0/24

  oci-sea-00                          198.18.4.0/24
  -----------------------------------------------------------------------

### Decision

The cluster uses node-specific /24 PodCIDRs within the 198.18.0.0/16 pod
network.

The existing PodCIDR assignments are part of the cluster\'s current
networking state.

### Consequences

-   Each node has an identifiable PodCIDR.

-   The unavailable euserv-eu-00 retains its previously assigned PodCIDR
    > in the Kubernetes inventory.

-   Pod IP addressing must not be confused with node InternalIP
    > addressing.

## ADR-008 --- FluxCD Controller Placement

**Status:** Accepted\
**Date:** 2026-08-26

### Context

Flux controllers currently run on oci-sea-00:

-   helm-controller

-   kustomize-controller

-   source-controller

Their common scheduling configuration is:

nodeSelector:

kubernetes.io/os: linux

tolerations:

\- key: node-role.kubernetes.io/control-plane

operator: Exists

effect: NoSchedule

There is no explicit nodeName, node affinity, or hostname selector in
their Deployment specifications.

### Decision

Flux controllers are permitted to run on Linux control-plane nodes by
virtue of tolerating the control-plane taint.

Their current placement on oci-sea-00 is therefore considered a valid
result of Kubernetes scheduling rather than an explicit hostname pinning
decision.

The Flux controllers are not considered statically bound to oci-sea-00.

### Consequences

-   Flux controller placement may change if scheduling conditions
    > change.

-   The Deployment configuration expresses eligibility rather than a
    > fixed node assignment.

-   Current pod placement should not be interpreted as hostname
    > affinity.

## ADR-009 --- System Workload Placement

**Status:** Accepted\
**Date:** 2026-08-26

### Context

Cluster-system workloads have different placement characteristics.

**Examples:**

-   kube-proxy runs on each active node through a DaemonSet.

-   Flannel runs on each active node through a DaemonSet.

-   CoreDNS currently runs on oci-sea-00.

-   Flux controllers tolerate control-plane taints.

-   CoreDNS additionally has preferred pod anti-affinity for other
    > kube-dns pods.

### Decision

System workloads are categorized according to their scheduling model:

**Node-local system services:**

DaemonSets such as kube-proxy and Flannel are expected to run on
applicable nodes.

**Cluster-level controllers:**

Components such as CoreDNS and Flux controllers are scheduled according
to their workload-specific selectors, tolerations, affinity and resource
requirements.

No assumption is made that every system component must run on every
node.

### Consequences

-   System workload placement is determined by the workload\'s
    > Kubernetes scheduling specification rather than by a blanket
    > \'system pods run on control-plane\' or \'system pods run on
    > workers\' rule.

## ADR-010 --- Resource-Constrained Node Policy

**Status:** Accepted\
**Date:** 2026-08-26

### Context

gcp-na-00 has significantly less allocatable memory than the other
active nodes:

-   **gcp-na-00:** \~873 MiB

-   **oci-sea-00:** \~11.55 GiB

-   **onprem-sea-00:** \~1.78 GiB

It also carries:

-   low-resource=true:NoSchedule

The node currently hosts only its control-plane/system components.

### Decision

gcp-na-00 is treated as a resource-constrained control-plane node.

The low-resource=true:NoSchedule taint is part of its scheduling policy.

### Consequences

-   Workloads must explicitly tolerate the low-resource taint to be
    > eligible for this node.

-   The node\'s control-plane role should not be interpreted as implying
    > general workload capacity.

-   Resource capacity must be considered independently from CPU
    > architecture or node role.

## ADR-011 --- Multi-Architecture Cluster

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The active cluster contains two CPU architectures:

amd64

└── gcp-na-00

arm64

├── oci-sea-00

└── onprem-sea-00

The Kubernetes version and container runtime are broadly aligned, but
architecture differs.

### Decision

The cluster is explicitly treated as a multi-architecture Kubernetes
cluster.

Workloads must be deployable on the architecture of nodes to which they
may be scheduled.

Where a workload is architecture-specific, its scheduling policy must
explicitly constrain it to compatible nodes.

### Consequences

-   Container image multi-architecture support is relevant to workload
    > portability.

-   Node architecture must be considered when interpreting scheduling
    > behavior.

-   A generic kubernetes.io/os=linux selector does not constrain CPU
    > architecture.

## ADR-012 --- Kubernetes Version Policy

**Status:** Accepted\
**Date:** 2026-08-26

### Context

Current Kubernetes versions are:

-   **gcp-na-00:** v1.35.1

-   **oci-sea-00:** v1.35.1

-   **onprem-sea-00:** v1.35.6

The control-plane nodes are therefore on the same Kubernetes version,
while the worker is on a later patch release.

### Decision

The cluster records Kubernetes version at node level and treats
control-plane version consistency as a distinct architectural concern
from worker versioning.

The current versions are:

-   **Control plane:** v1.35.1

-   **Worker:** v1.35.6

### Consequences

-   Version differences between nodes must be considered when evaluating
    > cluster behavior, upgrades, and compatibility.

## ADR-013 --- Container Runtime

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The active nodes use CRI-O:

-   **gcp-na-00:** cri-o://1.35.1

-   **oci-sea-00:** cri-o://1.35.1

-   **onprem-sea-00:** cri-o://1.35.5

### Decision

CRI-O is the container runtime used by the active Kubernetes nodes.

Runtime version is tracked independently per node.

### Consequences

-   Container runtime behavior is based on CRI-O.

-   Runtime version differences between control-plane and worker nodes
    > are part of platform inventory.

-   Runtime version should be included in future compatibility analysis.

## ADR-014 --- Node Lifecycle and Decommissioning

**Status:** Accepted\
**Date:** 2026-08-26

### Context

euserv-eu-00 remains represented as a Kubernetes Node but is
unavailable:

-   **Ready:** Unknown

-   **Unschedulable:** true

-   It carries unreachable and unschedulable taints and has no running
    > pods.

### Decision

Node lifecycle state is distinguished from the existence of a Kubernetes
Node object.

A node can be:

-   active and schedulable,

-   active but unschedulable,

-   unreachable/unavailable,

-   historical/decommissioned.

euserv-eu-00 is currently classified as unavailable.

### Consequences

-   Cluster inventory must not equate \'Node object exists\' with \'node
    > is operational.\'

## ADR-015 --- High Availability and Failure Model

**Status:** Accepted\
**Date:** 2026-08-26

### Context

The cluster has two active control-plane/etcd nodes in different
infrastructure locations:

GCP / NA

└── gcp-na-00

OCI / SEA

└── oci-sea-00

It also has a single active worker:

onprem / SEA

└── onprem-sea-00

A previously used external-etcd-only node exists in the inventory but is
unavailable.

### Decision

The cluster\'s HA model is defined separately for:

**Control plane:**

Two active control-plane nodes provide distributed control-plane
infrastructure.

**Worker capacity:**

Only one active general-purpose worker node currently exists. Therefore,
control-plane redundancy and workload-capacity redundancy are not
equivalent properties.

**Failure domains:**

Infrastructure location is treated as a meaningful failure domain
because active control-plane nodes are distributed across locations.

### Consequences

-   The cluster should not be described simply as \'HA\' without
    > specifying the component being discussed.

-   In particular: Control-plane redundancy does not imply redundant
    > application-worker capacity.

## ADR-016 --- GitOps Ownership Model

**Status:** Accepted\
**Date:** 2026-08-26

### Context

FluxCD is installed in flux-system with:

-   source-controller

-   kustomize-controller

-   helm-controller

The Deployments contain Flux ownership labels and Kustomize Toolkit
metadata:

kustomize.toolkit.fluxcd.io/name=flux-system

kustomize.toolkit.fluxcd.io/namespace=flux-system

### Decision

FluxCD is treated as the cluster\'s GitOps control mechanism for
resources under its management.

The Flux controllers are themselves part of the cluster\'s system
infrastructure and are scheduled according to their Kubernetes
Deployment policies.

### Consequences

-   GitOps-managed resources should be understood as declaratively
    > owned.

-   Manual changes to GitOps-managed resources may be reconciled by
    > Flux.

-   Flux controller placement is a scheduling concern separate from
    > GitOps ownership.

## ADR Dependency Map

ADR dependency map

The resulting architecture can now be read in this order:

ADR-001 Cluster Topology

│

├── ADR-002 Control Plane Placement

│ └── ADR-004 Scheduling Policy

│

├── ADR-005 etcd Topology

│ └── ADR-015 HA / Failure Model

│

├── ADR-003 Worker Placement

│

├── ADR-010 Resource Policy

│

└── ADR-011 Multi-Architecture

ADR-006 Networking

└── ADR-007 Pod CIDRs

ADR-009 System Workloads

└── ADR-008 Flux Placement

ADR-012 Kubernetes Versions

ADR-013 Container Runtime

ADR-014 Node Lifecycle

ADR-016 GitOps Ownership

One important property of this set: none of these ADRs claim that
something needs to be fixed merely because the inventory revealed it.
They capture the architecture and the decisions implied by the current
state.

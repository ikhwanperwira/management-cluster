# ADR-004 --- Node Taints, Labels and Scheduling Policy

**Status:** Accepted\
**Date:** 2026-08-26

## Context

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

## Decision

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

## Consequences

-   A pod\'s current node placement cannot be explained by node labels
    > alone. Its scheduling constraints must be inspected together with
    > node taints and available resources.

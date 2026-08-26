# ADR-009 --- System Workload Placement

**Status:** Accepted\
**Date:** 2026-08-26

## Context

Cluster-system workloads have different placement characteristics.

**Examples:**

-   kube-proxy runs on each active node through a DaemonSet.

-   Flannel runs on each active node through a DaemonSet.

-   CoreDNS currently runs on oci-sea-00.

-   Flux controllers tolerate control-plane taints.

-   CoreDNS additionally has preferred pod anti-affinity for other
    > kube-dns pods.

## Decision

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

## Consequences

-   System workload placement is determined by the workload\'s
    > Kubernetes scheduling specification rather than by a blanket
    > \'system pods run on control-plane\' or \'system pods run on
    > workers\' rule.

# ADR-003 --- Worker Placement and Workload Scheduling

**Status:** Accepted\
**Date:** 2026-08-26

## Context

onprem-sea-00 is the only active node designated exclusively as a
worker.

It has:

-   node-role.kubernetes.io/worker=

-   and no taints.

The active cluster currently has relatively few workload pods, while the
control-plane nodes host the Kubernetes control-plane and selected
system workloads.

## Decision

onprem-sea-00 is the cluster\'s dedicated general-purpose worker node.

Application workloads should be considered worker workloads by default.

Control-plane nodes are not considered general-purpose application
capacity unless a workload explicitly tolerates the relevant
control-plane taint.

## Consequences

-   Worker capacity is concentrated on onprem-sea-00.

-   Workload availability is inherently constrained by the number of
    > active worker nodes.

-   Pod placement must account for the fact that the worker and
    > control-plane pools are distinct.

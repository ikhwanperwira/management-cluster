# ADR-011 --- Multi-Architecture Cluster

**Status:** Accepted\
**Date:** 2026-08-26

## Context

The active cluster contains two CPU architectures:

amd64

└── gcp-na-00

arm64

├── oci-sea-00

└── onprem-sea-00

The Kubernetes version and container runtime are broadly aligned, but
architecture differs.

## Decision

The cluster is explicitly treated as a multi-architecture Kubernetes
cluster.

Workloads must be deployable on the architecture of nodes to which they
may be scheduled.

Where a workload is architecture-specific, its scheduling policy must
explicitly constrain it to compatible nodes.

## Consequences

-   Container image multi-architecture support is relevant to workload
    > portability.

-   Node architecture must be considered when interpreting scheduling
    > behavior.

-   A generic kubernetes.io/os=linux selector does not constrain CPU
    > architecture.

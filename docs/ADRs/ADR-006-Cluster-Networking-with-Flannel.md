# ADR-006 --- Cluster Networking with Flannel

**Status:** Accepted\
**Date:** 2026-08-26

## Context

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

## Decision

Flannel is the cluster\'s Kubernetes pod networking implementation.

The cluster uses per-node PodCIDR allocation with Flannel as the
networking layer.

## Consequences

-   Pod networking depends on Flannel being operational on each active
    > node.

-   Pod IPs are allocated from node-associated PodCIDRs.

-   Node networking and Pod networking are distinct from the nodes\'
    > 100.96.0.x internal addresses.

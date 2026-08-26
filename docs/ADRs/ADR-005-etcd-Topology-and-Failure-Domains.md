# ADR-005 --- etcd Topology and Failure Domains

**Status:** Accepted\
**Date:** 2026-08-26

## Context

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

## Decision

The active Kubernetes control-plane/etcd topology is recorded as:

-   **gcp-na-00**

-   **oci-sea-00**

euserv-eu-00 is recorded as an unavailable external-etcd-only node, not
as an active Kubernetes compute node.

The existence of the Kubernetes Node object for euserv-eu-00 is not
treated as evidence that the node is operational.

## Consequences

-   The active control-plane/etcd topology is explicitly documented.

-   euserv-eu-00 remains relevant to historical cluster state and
    > lifecycle discussions.

-   Any future analysis of etcd quorum or failure tolerance must
    > distinguish active etcd membership from Kubernetes Node objects.

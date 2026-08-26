# ADR-001 --- Kubernetes Cluster Topology

**Status:** Accepted\
**Date:** 2026-08-26

## Context

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

## Decision

The Kubernetes cluster is operated as a distributed, multi-location
Kubernetes cluster with:

-   **two active control-plane/etcd nodes:** gcp-na-00, oci-sea-00

-   **one active worker node:** onprem-sea-00

-   **one historical/unavailable external-etcd node:** euserv-eu-00

The node inventory and role assignments above are considered the
authoritative description of the current cluster topology.

## Consequences

-   Cluster architecture spans infrastructure providers and locations.

-   Control-plane and workload placement cannot be assumed to be
    > homogeneous.

-   CPU architecture must be considered when scheduling workloads.

-   The unavailable euserv-eu-00 must be distinguished from active
    > cluster nodes in architectural discussions.

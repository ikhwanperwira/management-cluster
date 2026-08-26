# ADR-015 --- High Availability and Failure Model

**Status:** Accepted\
**Date:** 2026-08-26

## Context

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

## Decision

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

## Consequences

-   The cluster should not be described simply as \'HA\' without
    > specifying the component being discussed.

-   In particular: Control-plane redundancy does not imply redundant
    > application-worker capacity.

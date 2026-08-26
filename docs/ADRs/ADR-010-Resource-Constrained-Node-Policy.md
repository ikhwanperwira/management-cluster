# ADR-010 --- Resource-Constrained Node Policy

**Status:** Accepted\
**Date:** 2026-08-26

## Context

gcp-na-00 has significantly less allocatable memory than the other
active nodes:

-   **gcp-na-00:** \~873 MiB

-   **oci-sea-00:** \~11.55 GiB

-   **onprem-sea-00:** \~1.78 GiB

It also carries:

-   low-resource=true:NoSchedule

The node currently hosts only its control-plane/system components.

## Decision

gcp-na-00 is treated as a resource-constrained control-plane node.

The low-resource=true:NoSchedule taint is part of its scheduling policy.

## Consequences

-   Workloads must explicitly tolerate the low-resource taint to be
    > eligible for this node.

-   The node\'s control-plane role should not be interpreted as implying
    > general workload capacity.

-   Resource capacity must be considered independently from CPU
    > architecture or node role.
